# VotingContract 컨트랙트
## 개요
이 컨트랙트는 투표를 수행하고 투표 결과를 기록하는 데 사용됩니다.

## 함수(Write)
> ### function vote(uint256 option)
> #### 설명: 사용자가 투표를 제출합니다.
> #### 파라미터:
> * option: uint256 - 투표 옵션(0~15)

> ### function retractVote() public
> #### 설명: 사용자가 이전에 제출한 투표를 취소합니다.

> ### function endRound() external onlyRole(DEFAULT_ADMIN_ROLE) 
> #### 설명: 현재 라운드의 투표를 종료하고 결과를 발표합니다.

> ### function uploadFinalResult(uint256 stakeRewardRate) external onlyRole(DEFAULT_ADMIN_ROLE)
> #### 설명: 최종 결과를 업로드하고, 스테이크 보상률을 설정합니다.
> #### 파라미터:
> * stakeRewardRate: uint256 - 설정할 스테이크 보상률

## 함수(Read)
> ### function getVotePower(address user) public view returns (uint256 totalAmount):
> #### 설명: 사용자의 투표 권한파워(스테이킹 중인 NFT수량)를 리턴합니다.
> #### 파라미터
> * user: address - 투표 권한을 조회할 사용자의 주소
> #### 리턴 파라미터:
> * totalAmount: uint256 - 사용자의 투표 권한

> ### function getVoterDetails(address voter) public view returns (uint256 option, uint256 power)
> #### 설명: 사용자가 투표한 옵션 및 투표파워를 리턴합니다.
> #### 파라미터
> * voter: address - 조회할 사용자의 주소
> #### 리턴 파라미터
> * option: uint256 _- 사용자의 투표 옵션
> * power: uint256 - 사용자의 투표 권한_

> ### function getVoteCountForOption(uint256 option) public view returns (uint256)
> #### 설명: 특정 투표 옵션에 대한 투표 파워를 리턴합니다.
> #### 파라미터
> * option: uint256 - 조회할 투표 옵션(0~15)
> #### 리턴 파라미터
> * voteCount: uint256 - 특정 투표 옵션에 대한 투표 수

> ### function getAllVoteCounts() public view returns (uint256[] memory)
> #### 설명: 모든 투표 옵션에 대한 투표 파워를 리턴합니다.
> #### 리턴 파라미터
> * totals: uint256[] - 각 투표 옵션에 대한 투표 수를 포함하는 배열

> ### function getCurrentRound() public view returns (uint256 round)
> #### 설명: 현재 라운드를 조회합니다.
> #### 파라미터 : 없음
> #### 리턴 파라미터
> * round: uint256 - 현재 라운드 번호

> ### function getStakeRewardRate() public view returns (uint256 stakeRewardRate)
> #### 설명: 현재 설정된 스테이크 보상률을 조회합니다.
> #### 파라미터 : 없음
> #### 리턴 파라미터
> * stakeRewardRate: uint256 - 현재 스테이크 보상률

## 이벤트
> ### event VoteCasted(uint256 indexed round, address indexed voter, uint256 option, uint256 power)
> #### 설명: 사용자가 투표를 제출했을 때 발생하는 이벤트입니다. 해당 라운드, 투표자, 투표 옵션 및 투표 파워가 기록됩니다.
> #### 파라미터
> * round: uint256 - 해당 라운드
> * voter: address - 투표자의 주소
> * option: uint256 - 투표 옵션
> * power: uint256 - 투표 권한

> ### event VoteRetracted(uint256 indexed round, address indexed voter, uint256 option, uint256 power)
> #### 설명: 사용자가 투표를 취소했을 때 발생하는 이벤트입니다. 해당 라운드, 투표자, 투표 옵션 및 취소된 투표 파워가 기록됩니다.
> #### 파라미터:
> * round: uint256 - 해당 라운드
> * voter: address - 투표자의 주소
> * option: uint256 - 취소된 투표 옵션
> * power: uint256 - 취소된 투표 권한

> #### event RoundEnded(uint256 round, uint256[] voteTotals)
> #### 설명: 라운드가 종료되었을 때 발생하는 이벤트로, 해당 라운드 및 각 투표 옵션에 대한 총 투표 파워가 기록됩니다.
> #### 파라미터:
> * round: uint256 - 종료된 라운드
> * voteTotals: uint256[] - 각 투표 옵션에 대한 총 투표 수를 포함하는 배열