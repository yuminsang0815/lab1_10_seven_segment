# 실험 전 레포트: 7세그먼트 디코더

작성일 2026-09-19.

## 목적과 예상값

4비트 2진 입력 `bcd[3:0]`에 대응하여 단일 7세그먼트 디스플레이에 16진수 문자 0~F를 표출하는 8비트 세그먼트 데이터 `seg_data[7:0]`를 생성합니다. top은 `seg_decoder`입니다. 출력 비트 순서는 `{a,b,c,d,e,f,g,dp}`이며, 1일 때 켜지는 active-high 방식을 적용하고 소수점 `dp`는 상시 소등(0)합니다.

| bcd[3:0] | 표시 문자 | seg_data[7:0] (16진) | 점등 세그먼트 | 비고 |
|---:|:---:|---|---|---|
| 0000 (0) | 0 | 8'hfc | a, b, c, d, e, f | g, dp 소등 |
| 0001 (1) | 1 | 8'h60 | b, c | 선분 2개 점등 |
| 1000 (8) | 8 | 8'hfe | a, b, c, d, e, f, g | 모든 문자 선분 점등 |
| 1111 (15) | F | 8'h8e | a, e, f, g | 16진 문자 표출 |

## 환경과 입력 파일

Windows, Git 2.55.0, VS Code 1.136.1, XSim 2026.1을 사용했습니다. 공개 템플릿의 커밋 `740aef5`를 공백이 있는 별도 폴더에 새로 clone했습니다. `LAB1.code-workspace`는 `vivado_2026_1/10_seven_segment`를 PROJECT, `common`을 COMMON으로 연결합니다.

[RTL](../../src/seg_decoder.v), [TB](../../sim/tb_seg_decoder.sv), [XDC](../../constraints/seg_decoder.xdc)를 사용했습니다. TB top은 `tb_seg_decoder`이며 루프 변수 n을 0부터 15까지 10ns 간격으로 인가하여 16가지 전수 입력을 검증합니다[cite: 8]. 폰트 패턴 배열 기대값과 실제 출력을 비교하여 불일치 시 `$fatal`로 종료하고, 16건 완료 시 PASS 출력 후 160ns에 종료합니다.

## 실행과 관찰

VS Code 새 창에서 workspace를 열고 slang-server와 VaporView를 활성화했습니다. 저장 후 Terminal → Run Task...의 Check tools, Simulate를 실행하고 Open waveform으로 VCD를 열었습니다. `bcd,seg_data`를 추가하고 Zoom to Fit와 ns 단위를 선택했습니다.

실제 [VS Code 작업 실행 로그](../../evidence/10/vscode/simulation.txt)에서 `LAB1_PASS seg_decoder cases=16`과 160ns 종료를 확인했습니다.

| 구간(ns) | bcd (hex) | seg_data (hex) | 해석 |
|---|---|---|---|
| 0–10 | 0 | fc | 숫자 0 폰트 (a~f 켜짐) |
| 10–20 | 1 | 60 | 숫자 1 폰트 (b, c 켜짐) |
| 80–90 | 8 | fe | 숫자 8 폰트 (a~g 전체 켜짐) |
| 150–160 | F | 8e | 문자 F 폰트 (a, e, f, g 켜짐) |

전환 순간 대신 구간 중간에서 커서를 읽었습니다. 파형만 눈으로 맞아 보이는 것에 더해 자동 비교 16건과 정상 종료를 함께 확인했습니다.

## 보드 실험 계획

DIP1\~4(bcd[3:0]), 7-Segment(seg_data[7:0]), LVCMOS33입니다. Vivado에서 같은 TB를 실행한 다음 bit를 생성하고 보드에 다운로드합니다. DIP 스위치를 0(0000), 8(1000), F(1111) 등으로 조작하면서 FND 상에 16진수 문자가 또렷하고 정확하게 표시되는 장면을 사진·영상에 담을 계획입니다.