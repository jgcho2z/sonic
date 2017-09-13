## 개요 
* 본 문서는 지능형 콜센터 프로젝트에서 Call Agent가 제공하는 DTMF 검출 기능에 대한 아이페이지온 자체 시험 결과서 입니다.
* Call Agent는 Dtmf 수집에 있어서, 아래와 같이 DtmfParam을 정의하였습니다.
* 아래의 파라미터는 Session Agent가 CallAgentThriftInterface 의 아래 Api를 호출할 때, Call Agent로 전달됩니다.

```
   i32 getDtmfPlayWav(1:string sessionId, 2:i64 requestNo, 3:i64 wavId, 4:DtmfParam param),
   i32 getDtmfPlayTts(1:string sessionId, 2:i64 messageNo, 3:DtmfParam param),
```

* Call Agent는 각각의 파라미터의 값에 정의된 대로 동작하며, 사용자가 입력하는 DTMF를 수집합니다.
* 그리고 아래의 SessionAgentThriftInterface의 api를 통하여 그 결과를 전달합니다.

```
   i32 sendDtmfResult(1:string sessionId, 2:i32 resultCode, 3:list<byte> dtmfData),
```

* 정의된 DtmfParam은 아래와 같습니다.

```
	struct DtmfParam
	{
	  1:i64 reprompt,
	  2:i64 noDigitsReprompt,
	  3:i64 failAnnouncement,
	  4:i64 successAnnouncement,
	  5:bool nonInterruptiblePlay,
	  6:i32 maxDigits,
	  7:i32 minDigits,
	  8:string digitPattern,
	  9:i32 firstDigitTimer,
	  10:i32 extraDigitTimer,
	  11:string startInputKey,
	  12:string endInputKey,
	  13:i32 numberOfAttempts
	}
```
### Dtmf Parameter 정의
* reprompt 사용자로부터 유효한 디지트 또는 음성의 입력에 실패한 경우 송출되는 안내방송.<br>Reprompt 가 정의되지 않은 경우 디폴트 값은 Initial prompt 이다. ex)  “다시 입력해 주십시오.”
* no digits reprompt PC 이벤트 실행중에 사용자로부터 유효한 디지트의 입력에 실패한 경우 송출되는 안내방송, <br> “번호가 입력되지 않았습니다. 번호를 입력해 주십시오.”
* failure announcement 모든 사용자 입력 시도가 실패했을 때 송출되는 안내방송 <br> “지금은 연결할 수 없습니다. 잠시 후 다시 걸어 주십시오.”
* success announcement 데이터 수집이 성공했을 때 송출되는 안내방송 <br> “안내원과 통화를 연결 중입니다. 잠시만 기다려 주십시오.”
* non-interruptible play ni=true 일 경우, Initial Prompt는 어떤 음성 또는 디지트에 의해 중단되지 않는다.  <br> Defaults to false
* maximum of digits 수집되어야 할 최대 디지트 수 Defaults to one (1 디지트),<br> Digit Pattern 파라미터가 사용될 경우, mx 파라미터를 사용하면 안된다.
* minimum of digits 수집되어야 할 최소 디지트 수 Defaults to one (1 디지트),<br> Digit Pattern 파라미터가 사용될 경우, mn 파라미터를 사용하면 안된다.
* digit pattern 수집되어야 할 유효한 디지트 맵. <br> mx 또는 mn 파라미터가 사용될 경우, dp 파라미터를 사용하면 안된다.
* first digit timer 처음 digit를 입력하는데 유저에게 허락되어진 시간을 의미한다. <br> 단위 100ms Default to 50 (5 seconds).
* extra digit timer 기대되어지는 digit가 입력되었을 때, final digit가 입력되는데 주어지는 시간 일반적으로 terminating key를 위해 사용되어짐.  <br> 단위 100ms 설정되지 않으면 활성화 되지 않음
* start input key First digit가 수집 되어졌을 때 허용되는 keys인지 결정 The default key set is 0-9
* end input key Digit 수집을 끝내는 것을 나타내는 key이다.  The default end input key is the # key
* number of attempts : 재시도 가능 횟수 Defaults to 1

## 시험 환경
### 시험 멘트
---
* initial prompt : 비밀번호 네자리를 눌러 주십시오. (7013)
* reprompt : 잘못 누르셨습니다. 다시 한번 비밀번호 네자리를 눌러 주십시오. (7014)
* failAnnouncement : 잘못 누르셨습니다. 다음에 다시 시도해 주십시오. (7015)
* successAnnouncement : 잘 누르셨습니다. 감사합니다. (7016)
* noDigitReprompt : 번호가 입력되지 않았습니다. 비밀번호 네자리를 눌러 주십시오. (7017)

### 시험 시뮬레이터
---
* Dtmf 시험을 위하여 아래와 같은 시험 환경을 구성하고, 시뮬레이터를 작성하였습니다.
#### Session Agent Simulator 
* 연결된 호에 대하여 Session Agent처럼 동작합니다.
* command를 입력 받아 필요로 하는 CallAgentThriftInterface api를 호출합니다.
* 아래와 같이 DtmfParam을 각각 입력할 수 있는 command를 제공합니다.
* DtmfParam의 type과 value를 입력하면, global instance에 값이 입력됩니다.
* getDtmfPlayWav나 getDtmfPlayTts를 호출하면, global DtmfParam instance에 입력한 값으로 해당 api가 호출됩니다.

```
SA > help
help : show this help string
ping : check ping
getCallSessionList : request getCallSessionList
getCallSessionInfo : request getCallSessionInfo arg[SessionId]
quit : exit
set first Dtmf attributes, and run getDtmfPlayWav, getDtmfPlayTts : to test DtmfParam
ex)
 CLI> reprompt 7001
 CLI> noDigitsReprompt 7002
 CLI> failAnnouncement 7002
 CLI> getDtmfPlayTts

class DtmfParam:
  Attributes:
   - reprompt
   - noDigitsReprompt
   - failAnnouncement
   - successAnnouncement
   - nonInterruptiblePlay
   - maxDigits
   - minDigits
   - digitPattern
   - firstDigitTimer
   - extraDigitTimer
   - startInputKey
   - endInputKey
   - numberOfAttempts
```

#### TTS simulator
* 현재는 실제 TTS와 연동되지는 않고, 준비된 멘트를 송출합니다.

#### SATRIB Log
* Call Agent의 Session Agent 연동 블록으로 실제 송수신된 메세지를 로그로 확인합니다.

#### MCST Log
* Call Agent의 MPM 모듈에서 실제 음성 play를 담당하는 연동 블록으로 송출된 MENT를 로그로 확인합니다.
* 해당 블록의 로그는 너무 많으므로 중요한 내용만을 grep하여 첨부합니다.

```
	 tail -F MCST_0.log | egrep "evtName.pc|DIGIT|ivrStartPlay.SI.....Med|oc.rc.....dc=|of.rc="
```

## 시험 진행
* 각 파라미터가 시뮬레이터에 의해 Call Agent로 제대로 전달되고, 동작함을 확인합니다.
* getDtmfPlayTts와 getDtmfPlayWav는 initialPrompt가 wav인지 Tts인지만 다르므로 getDtmfPlayTts는 기본 시험만 진행하고 생략한다.

### getDtmfPlayWav
---
* 아무 파라미터가 정의되지 않았을 경우의 getDtmfPlayWav의 동작을 확인합니다.
#### 결과
* initial prompt(7001) 를 송출하고, dtmf(1)를 수집 했습니다.
#### thrift 연동 로그
```
[21:31:53.3628] TX notifyIncommingCall   CallerNum[07070001004] SAG|SA[1|2] tenantId[T0001] siteId[S0001] callKey[iJIOAAMAYFgAALIXrBAXIQ--c15@ipageon.com] callerNumber[07070001004] calleeNumber[23589] mediaIp[172.16.23.183] mediaPort[25000]
[21:31:53.4803] TX openCallSession       CallerNum[07070001004] SAG|SA[1|2] tenantId[T0001] siteId[S0001] callKey[iJIOAAMAYFgAALIXrBAXIQ--c15@ipageon.com] callerNumber[07070001004] calleeNumber[23589] mediaIp[172.16.23.183] mediaPort[25000]
[21:31:53.4811] RX openCallSessionResult CallerNum[07070001004] SAG|SA[1|2] resultCode[1] callKey[iJIOAAMAYFgAALIXrBAXIQ--c15@ipageon.com] sessionId[SM1_20170831_DUMMY_1] sttIp[172.16.23.183] sttPort[1]
[21:31:53.5045] TX notifyReadyForMediaTransfer  CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1]
[21:31:53.5052] RX playCallWav           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0] repeatOption[1] wavIds-size[2]
[21:31:53.5125] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0]
[21:31:53.5131] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[21:31:59.0402] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0]
[21:32:10.0040] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[1] wavId[7001] DtmfParam[reprompt|0,noDigitsReprompt|0,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|0,minDigits|0,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|0]
[21:32:10.0323] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[1]
[21:32:10.0330] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[21:32:14.2150] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[1]
[21:32:14.2174] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[1]
```
#### MCST 동작 로그
```
0913 21:32:10.0297 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(0) evtName:pc, evtPara:ip=7001[Lang=kor] ni=false dfmRecordId:0 requestNo[1]
0913 21:32:10.0306 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7001)
0913 21:32:10.0312 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(0) aid(1) svc(wav) evtName(pc)
0913 21:32:14.2137 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI000] dgt INBAND DTMF DIGIT (1)
0913 21:32:14.2142 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH000] oc/rc=100 dc=1 na=1
0913 21:32:14.2200 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=1 na=1)
```

### getDtmfPlayTts
---
* 아무 파라미터가 정의되지 않았을 경우의 getDtmfPlayTts의 동작확인합니다. 
#### 결과
* tts ment 를 송출하고, dtmf(2)를 수집하였습니다.
#### thrift 연동 로그
```
[21:38:33.1929] TX notifyIncommingCall   CallerNum[07070001004] SAG|SA[1|2] tenantId[T0001] siteId[S0001] callKey[FiAFAAMAcHEAALIXrBAXIQ--c17@ipageon.com] callerNumber[07070001004] calleeNumber[23589] mediaIp[172.16.23.183] mediaPort[25008]
[21:38:33.3021] TX openCallSession       CallerNum[07070001004] SAG|SA[1|2] tenantId[T0001] siteId[S0001] callKey[FiAFAAMAcHEAALIXrBAXIQ--c17@ipageon.com] callerNumber[07070001004] calleeNumber[23589] mediaIp[172.16.23.183] mediaPort[25008]
[21:38:33.3029] RX openCallSessionResult CallerNum[07070001004] SAG|SA[1|2] resultCode[1] callKey[FiAFAAMAcHEAALIXrBAXIQ--c17@ipageon.com] sessionId[SM1_20170831_DUMMY_1] sttIp[172.16.23.183] sttPort[1]
[21:38:33.3225] TX notifyReadyForMediaTransfer  CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1]
[21:38:33.3232] RX playCallWav           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0] repeatOption[1] wavIds-size[2]
[21:38:33.3286] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0]
[21:38:33.3290] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[21:38:38.8501] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0]
[21:38:40.0473] RX getDtmfPlayTts        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] messageNo[1] DtmfParam[reprompt|0,noDigitsReprompt|0,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|0,minDigits|0,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|0]
[21:38:40.0543] TX notifyPlayCallTts     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] messageNo[1]
[21:38:40.0552] RX resumeCallSendStt     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1]
[21:38:43.0620] TX notifyStopCallTts     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] messageNo[1]
[21:38:43.0643] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[2]

```
#### MCST 동작 로그
```
0913 21:38:33.3265 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI002] MediaType(1) FileType (14) aID(7001)
0913 21:38:36.2503 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI002] MediaType(1) FileType (14) aID(7002)
0913 21:38:40.0494 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(2) evtName:pc, evtPara:ip=ts(1)[Lang=kor] ni=false dfmRecordId:0 requestNo[1]
0913 21:38:40.0537 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(2) aid(1) svc(tts) evtName(pc)
0913 21:38:43.0604 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI002] dgt INBAND DTMF DIGIT (2)
0913 21:38:43.0613 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH002] oc/rc=100 dc=2 na=1
0913 21:38:43.0665 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=2 na=1)
0913 21:38:43.0673 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(2) evtName:pc, evtPara:ip=ts(2)[Lang=kor] ni=false dfmRecordId:0 requestNo[2]
0913 21:38:43.0701 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(2) aid(2) svc(tts) evtName(pc)
0913 21:39:04.4210 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(2) aid(2) svc(tts) evtName(pc)


```

### numberOfAttempts 3, mindigits 2, reprompt 시험
---
* na를 3, minDigit 3, maxDigits 5  ,reprompt 설정하고 시험합니다. 
* reprompt : 잘못 누르셨습니다. 다시 한번 비밀번호 네자리를 눌러 주십시오. (7014)
#### 결과
* 멘트 play가 종료되고도 DTMF를 '1'한개만 누른 후, 대기, 7014 멘트가 play됨을 확인하였습니다.
* 2번 플레이되고, 330 "Max attempts exceeded" 가 전달됨을 확인했습니다.

#### thrift 연동 로그
```
[22:19:13.3215] TX notifyIncommingCall   CallerNum[07070001004] SAG|SA[1|2] tenantId[T0001] siteId[S0001] callKey[0YcDAAMA8AkAALIXrBAXIQ--c18@ipageon.com] callerNumber[07070001004] calleeNumber[23589] mediaIp[172.16.23.183] mediaPort[25020]
[22:19:13.4369] TX openCallSession       CallerNum[07070001004] SAG|SA[1|2] tenantId[T0001] siteId[S0001] callKey[0YcDAAMA8AkAALIXrBAXIQ--c18@ipageon.com] callerNumber[07070001004] calleeNumber[23589] mediaIp[172.16.23.183] mediaPort[25020]
[22:19:13.4376] RX openCallSessionResult CallerNum[07070001004] SAG|SA[1|2] resultCode[1] callKey[0YcDAAMA8AkAALIXrBAXIQ--c18@ipageon.com] sessionId[SM1_20170831_DUMMY_1] sttIp[172.16.23.183] sttPort[1]
[22:19:13.4572] TX notifyReadyForMediaTransfer  CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1]
[22:19:18.3220] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0] wavId[7013] DtmfParam[reprompt|7014,noDigitsReprompt|0,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|5,minDigits|3,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[22:19:18.3277] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0]
[22:19:18.3281] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[22:19:52.0223] TX reportError           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] functionName[getDtmfPlayWav] errorCode[330]
```
#### MCST 동작 로그
```
0913 22:19:18.3234 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(5) evtName:pc, evtPara:ip=7013[Lang=kor] rp=7014[Lang=kor] ni=false mn=3 mx=5 na=3 dfmRecordId:0 requestNo[0]
0913 22:19:18.3256 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI005] MediaType(1) FileType (14) aID(7013)
0913 22:19:18.3272 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(5) aid(0) svc(wav) evtName(pc)
0913 22:19:21.1241 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI005] dgt INBAND DTMF DIGIT (1)
0913 22:19:21.1244 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI005] dgt INBAND DTMF DIGIT (1)
0913 22:19:24.2224 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI005] MediaType(1) FileType (14) aID(7014)
0913 22:19:38.0220 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI005] MediaType(1) FileType (14) aID(7014)
0913 22:19:52.0218 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(5) aid(0) svc(wav) evtName(pc)
0913 22:19:52.0247 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (of/rc=330 na=3)
```

### numberOfAttempts 3, nodigitReprompt 시험
---
* na를 3, noDigitReprompt 설정하고 시험합니다. 
* noDigitReprompt : 번호가 입력되지 않았습니다. 비밀번호 네자리를 눌러 주십시오. (7017)
#### 결과
* 멘트 play가 종료되고도 DTMF를 누르지 않고 대기, 7017 멘트가 play됨을 확인하였습니다.
* 2번 플레이되고, 330 "Max attempts exceeded" 가 전달됨을 확인했습니다.
#### thrift 연동 로그
```
[22:30:11.1140] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[2] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|3,minDigits|3,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[22:30:11.1195] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[2]
[22:30:11.1200] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[22:30:49.4224] TX reportError           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] functionName[getDtmfPlayWav] errorCode[330]
```
#### MCST 동작 로그
```
0913 22:30:11.1153 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(7) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] ni=false mn=3 mx=3 na=3 dfmRecordId:0 requestNo[2]
0913 22:30:11.1175 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7013)
0913 22:30:11.1190 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(7) aid(2) svc(wav) evtName(pc)
0913 22:30:22.2224 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7017)
0913 22:30:35.8217 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7017)
0913 22:30:49.4216 0 0 (AuUtil.c      ,  457) DEB1 [auSendOperationFail] [auSendOperationFail CH007] of/rc=330 na=3
0913 22:30:49.4218 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(7) aid(2) svc(wav) evtName(pc)
0913 22:30:49.4249 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (of/rc=330 na=3)
```

### numberOfAttempts 3, failAnnouncement 시험
---
* na를 3, failAnnouncement 설정하고 시험합니다. 
* failAnnouncement : 잘못 누르셨습니다. 다음에 다시 시도해 주십시오. (7015)
#### 결과
* 멘트 play가 종료되고도 DTMF를 누르지 않고 대기,
* 7017 멘트가 2번 더 play됨을 확인하였습니다.
* 이후 7015 멘트가 play됨을 확인하였습니다.
* 2번 플레이되고, 330 "Max attempts exceeded" 가 전달됨을 확인했습니다.
#### thrift 연동 로그
```
[22:33:05.7819] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[3] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|7015,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|3,minDigits|3,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[22:33:05.7879] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[3]
[22:33:05.7884] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[22:33:47.9142] TX reportError           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] functionName[getDtmfPlayWav] errorCode[330]
```
#### MCST 동작 로그
```
0913 22:33:05.7834 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(7) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] fa=7015[Lang=kor] ni=false mn=3 mx=3 na=3 dfmRecordId:0 requestNo[3]
0913 22:33:05.7858 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7013)
0913 22:33:05.7874 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(7) aid(3) svc(wav) evtName(pc)
0913 22:33:16.9224 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7017)
0913 22:33:30.5219 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7017)
0913 22:33:44.1215 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7015)
0913 22:33:47.9134 0 0 (AuUtil.c      ,  457) DEB1 [auSendOperationFail] [auSendOperationFail CH007] of/rc=330 na=3
0913 22:33:47.9137 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(7) aid(3) svc(wav) evtName(pc)
0913 22:33:47.9164 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (of/rc=330 na=3)
```

### numberOfAttempts 3, nonInterruptiblePlay 시험
---
* na를 3, nonInterruptiblePlay 설정하고 시험합니다. 
#### 결과
* digit을 눌러도 멘트가 끊어지지 않음을 확인하였습니다.
#### thrift 연동 로그
```
[22:39:01.4040] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[6] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|7015,successAnnouncement|0,nonInterruptiblePlay|True,maxDigits|3,minDigits|3,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[22:39:01.4102] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[6]
[22:39:01.4108] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[22:39:04.5833] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[6]
[22:39:04.5856] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[111]
```
#### MCST 동작 로그
```
0913 22:39:01.4054 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(7) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] fa=7015[Lang=kor] ni=true mn=3 mx=3 na=3 dfmRecordId:0 requestNo[6]
0913 22:39:01.4080 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI007] MediaType(1) FileType (14) aID(7013)
0913 22:39:01.4097 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(7) aid(6) svc(wav) evtName(pc)
0913 22:39:02.3025 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:02.3026 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:03.0025 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:03.0026 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:03.7022 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:03.7026 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:04.5822 0 0 (IVRagtAt.c    ,   72) DEB0 [cbIVRagtAt] [cbIVRagtAt SI007] dgt INBAND DTMF DIGIT (1)
0913 22:39:04.5827 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH007] oc/rc=100 dc=111 na=1
0913 22:39:04.5864 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=111 na=1)
```

### maxDigits 4 시험
---
* maxDigits 을 4로 설정하고 시험합니다.
#### 결과
* dtmf를 1234567890을 누르고, 1234가 수집됨을 확인하였습니다.
#### thrift 연동 로그
```
[22:44:40.4430] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] requestNo[2] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|7015,successAnnouncement|0,nonInterruptiblePlay|True,maxDigits|4,minDigits|3,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[22:44:40.4491] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] requestNo[2]
[22:44:40.4497] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] timeoutMs[10000]
[22:44:44.4819] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] requestNo[2]
[22:44:44.4842] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] resultCode[0] dtmfData[1234]
```
#### MCST 동작 로그
```
0913 22:44:40.4443 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(9) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] fa=7015[Lang=kor] ni=true mn=3 mx=4 na=3 dfmRecordId:0 requestNo[2]
0913 22:44:40.4470 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI009] MediaType(1) FileType (14) aID(7013)
0913 22:44:40.4486 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(9) aid(2) svc(wav) evtName(pc)
0913 22:44:44.4811 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH009] oc/rc=100 dc=1234 na=1
0913 22:44:44.4850 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=1234 na=1)
```

### minDigits 2, maxDigit 4 시험
---
* minDigits 를 2로 설정하고 시험합니다.
* maxDigits 을 4로 설정하고 시험합니다.
* dtmf를 "12" 만 누르고 대기합니다.
#### 결과
* "12"가 수집됨을 확인하였습니다.
#### thrift 연동 로그
```
[22:45:57.2430] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] requestNo[4] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|7015,successAnnouncement|0,nonInterruptiblePlay|True,maxDigits|4,minDigits|2,digitPattern|,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[22:45:57.2493] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] requestNo[4]
[22:45:57.2498] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] timeoutMs[10000]
[22:46:19.4222] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] requestNo[4]
[22:46:19.4248] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_3] resultCode[0] dtmfData[12]
```
#### MCST 동작 로그
```
0913 22:45:57.2445 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(9) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] fa=7015[Lang=kor] ni=true mn=2 mx=4 na=3 dfmRecordId:0 requestNo[4]
0913 22:45:57.2471 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI009] MediaType(1) FileType (14) aID(7013)
0913 22:45:57.2487 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(9) aid(4) svc(wav) evtName(pc)
0913 22:46:08.3219 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI009] MediaType(1) FileType (14) aID(7017)
0913 22:46:19.4213 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH009] oc/rc=100 dc=12 na=2
0913 22:46:19.4258 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=12 na=2)
```

### digitPattern 12340 시험
---
* digitPattern을 1|2|3|4|0으로 설정하고 시험합니다.
* dtmf를 "5"을 입력합니다.
* 다시 dtmf를 "3"을 입력합니다.

#### 결과
* 329 Digit pattern not matched가 리턴됨을 확인하였습니다.
* 3이 수집됨을 확인하였습니다. 
#### thrift 연동 로그
* 첫번째 케이스
```
[22:53:35.9330] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[4] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|0,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|0,minDigits|0,digitPattern|1|2|3|4|0,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|0]
[22:53:35.9384] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[4]
[22:53:35.9389] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[22:53:37.7641] TX reportError           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] functionName[getDtmfPlayWav] errorCode[329]
```
* 두번째 케이스
```
[[22:54:43.7640] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[5] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|0,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|0,minDigits|0,digitPattern|1|2|3|4|0,firstDigitTimer|0,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|0]
[22:54:43.7696] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[5]
[22:54:43.7702] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[22:54:45.1113] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[5]
[22:54:45.1136] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[1]
```
#### MCST 동작 로그
* 첫번째 케이스
```
0913 22:53:35.9343 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(10) evtName:pc, evtPara:ip=7013[Lang=kor] ni=false dp=1|2|3|4|0 dfmRecordId:0 requestNo[4]
0913 22:53:35.9363 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI010] MediaType(1) FileType (14) aID(7013)
0913 22:53:35.9379 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(10) aid(4) svc(wav) evtName(pc)
0913 22:53:37.7633 0 0 (AuUtil.c      ,  457) DEB1 [auSendOperationFail] [auSendOperationFail CH010] of/rc=329 na=1
0913 22:53:37.7635 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(10) aid(4) svc(wav) evtName(pc)
0913 22:53:37.7663 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (of/rc=329 na=1)
```
* 두번째 케이스
```
0913 22:54:43.7654 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(10) evtName:pc, evtPara:ip=7013[Lang=kor] ni=false dp=1|2|3|4|0 dfmRecordId:0 requestNo[5]
0913 22:54:43.7675 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI010] MediaType(1) FileType (14) aID(7013)
0913 22:54:43.7691 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(10) aid(5) svc(wav) evtName(pc)
0913 22:54:45.1106 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH010] oc/rc=100 dc=1 na=1
0913 22:54:45.1144 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=1 na=1)
```

### firstDigitTimer 20, noDigitsReprompt 시험
---
* firstDigitTimer를 20으로 입력합니다. 
* noDigitReprompt를 설정합니다.
* noDigitReprompt : 번호가 입력되지 않았습니다. 비밀번호 네자리를 눌러 주십시오. (7017)
#### 결과
* 2초가 경과된 후, 7017 멘트가 play되는 것을 확인하였습니다. 
#### thrift 연동 로그
```
[23:33:09.2630] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|0,minDigits|0,digitPattern|,firstDigitTimer|20,extraDigitTimer|0,startInputKey|,endInputKey|,numberOfAttempts|3]
[23:33:09.2724] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[0]
[23:33:09.2732] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
n[23:33:26.5461] TX reportError           CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] functionName[getDtmfPlayWav] errorCode[330]
```
#### MCST 동작 로그
```
0913 23:33:09.2686 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(0) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] ni=false fdt=20 na=3 dfmRecordId:0 requestNo[0]
0913 23:33:09.2702 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7013)
0913 23:33:09.2719 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(0) aid(0) svc(wav) evtName(pc)
0913 23:33:13.3464 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7017)
0913 23:33:19.9464 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7017)
0913 23:33:26.5453 0 0 (AuUtil.c      ,  457) DEB1 [auSendOperationFail] [auSendOperationFail CH000] of/rc=330 na=3
0913 23:33:26.5455 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(0) aid(0) svc(wav) evtName(pc)
0913 23:33:26.5484 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (of/rc=330 na=3)
```

### extraDigitTimer 20, minDigits 2, maxDigits 5 시험
---
* extraDigitTimer 를 20으로 입력합니다.
* mindigits 를 2로 입력합니다.
* maxdigits 를 5로 입력합니다.
#### 결과
* "123"을 입력하고 2초가 지난 후, sendDtmfResult가 전달됨을 확인하였습니다.
#### thrift 연동 로그
```
[23:34:44.5140] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[1] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|5,minDigits|2,digitPattern|,firstDigitTimer|20,extraDigitTimer|20,startInputKey|,endInputKey|,numberOfAttempts|3]
[23:34:44.5199] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[1]
[23:34:44.5206] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[23:34:51.1461] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[1]
[23:34:51.1486] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[123]
```
#### MCST 동작 로그
```
0913 23:34:44.5156 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(0) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] ni=false mn=2 mx=5 fdt=20 edt=20 na=3 dfmRecordId:0 requestNo[1]
0913 23:34:44.5179 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7013)
0913 23:34:44.5195 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(0) aid(1) svc(wav) evtName(pc)
0913 23:34:51.1452 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH000] oc/rc=100 dc=123 na=1
0913 23:34:51.1493 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=123 na=1)
```

### startInputKey * 시험
---
* startInputKey를 "*"로 설정하였습니다.
* minDigits 를 3, maxDigit을 5으로 설정하였습니다.
#### 결과
* "23*45678"을 순차적으로 입력하고, *4567이 수집됨을 확인하였습니다.
#### thrift 연동 로그
```
[23:36:14.5420] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[3] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|5,minDigits|2,digitPattern|,firstDigitTimer|20,extraDigitTimer|20,startInputKey|*,endInputKey|,numberOfAttempts|3]
[23:36:14.5483] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[3]
[23:36:14.5489] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[23:36:20.1931] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[3]
[23:36:20.1953] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[*4567]
```
#### MCST 동작 로그
```
0913 23:36:14.5434 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(0) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] ni=false mn=2 mx=5 fdt=20 edt=20 sik=* na=3 dfmRecordId:0 requestNo[3]
0913 23:36:14.5458 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7013)
0913 23:36:14.5478 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(0) aid(3) svc(wav) evtName(pc)
0913 23:36:20.1924 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH000] oc/rc=100 dc=*4567 na=1
0913 23:36:20.1960 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=*4567 na=1)
```

### endInputKey 9 시험
---
* startInputKey를 "*"로 설정하였습니다.
* endInputKey를 9로 설정하였습니다.
* minDigits 를 3, maxDigit을 5으로 설정하였습니다.
#### 결과
* "123*456978"을 순차적으로 입력하고, "*456" 이 수집됨을 확인하였습니다.
#### thrift 연동 로그
```
[23:39:56.9550] RX getDtmfPlayWav        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[8] wavId[7013] DtmfParam[reprompt|0,noDigitsReprompt|7017,failAnnouncement|0,successAnnouncement|0,nonInterruptiblePlay|False,maxDigits|5,minDigits|2,digitPattern|,firstDigitTimer|20,extraDigitTimer|20,startInputKey|*,endInputKey|9,numberOfAttempts|3]
[23:39:56.9614] TX notifyPlayCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[8]
[23:39:56.9620] RX pauseCallSendStt      CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] timeoutMs[10000]
[23:40:01.3653] TX notifyStopCallWav     CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] requestNo[8]
[23:40:01.3676] TX sendDtmfResult        CallerNum[07070001004] SAG|SA[1|2] sessionId[SM1_20170831_DUMMY_1] resultCode[0] dtmfData[*456]
```
#### MCST 동작 로그
```
0913 23:39:56.9565 0 0 (McstTerm.c    ,  925) DEB0 [mExecAuPkg] @ [mExecAuPkg] CH(0) evtName:pc, evtPara:ip=7013[Lang=kor] nd=7017[Lang=kor] ni=false mn=2 mx=5 fdt=20 edt=20 sik=* eik=9 na=3 dfmRecordId:0 requestNo[8]
0913 23:39:56.9590 0 0 (IVRagtAu.c    , 1923) DEB2 [ivrStartPlay] [ivrStartPlay SI000] MediaType(1) FileType (14) aID(7013)
0913 23:39:56.9608 0 0 (McstTerm.c    , 1482) DEB0 [mNotifyStartPlay] @TtsMent notify START_PLAY mSessId(0) aid(8) svc(wav) evtName(pc)
0913 23:40:01.3647 0 0 (AuUtil.c      ,  526) DEB2 [auSendOperationOk] [auSendOperationOk CH000] oc/rc=100 dc=*456 na=1
0913 23:40:01.3682 0 0 (McstTerm.c    , 1592) DEB0 [mSendRsAuPkg] Send MCS_rsAuPkg (oc/rc=100 dc=*456 na=1)
```


