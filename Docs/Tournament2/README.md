# MiracleTournamentManager v2.0.0

이 Solidity 스마트 컨트랙트는 토너먼트 생성과 관리에 필요한 기능을 제공합니다. 참가자 등록, 참가비, 상금 분배, 수수료(Developer, WinnerClub, Platform) 설정 및 배분, 우승자 지급 등 다양한 기능을 제공합니다.

## 목차

1. [개요](#개요)  
2. [주요 기능](#주요-기능)  
3. [역할(Role) 및 권한(Permissions)](#역할role-및-권한permissions)  
4. [사용 방법](#사용-방법)  
    - [토너먼트 생성](#토너먼트-생성)  
    - [토너먼트 참가](#토너먼트-참가)  
    - [참가자 제거](#참가자-제거)  
    - [참가자 무작위 셔플](#참가자-무작위-셔플)  
    - [토너먼트 취소](#토너먼트-취소)  
    - [토너먼트 종료 (수동-청구 방식)](#토너먼트-종료-수동-청구-방식)  
    - [토너먼트 종료 (자동 분배 방식)](#토너먼트-종료-자동-분배-방식)  
    - [상금 청구 (수동-청구 방식)](#상금-청구-수동-청구-방식)  
5. [수수료 관리](#수수료-관리)  
6. [조회 함수 (View Functions)](#조회-함수-view-functions)  

---

## 개요

`MiracleTournamentManager`는 아래와 같은 토너먼트 관련 기능을 제공합니다.

- **상금 풀(Prize Pool)**: 상금용 토큰을 사전에 예치.
- **참가비(Entry Fee)**: 토너먼트 참가 시 일정 토큰(ERC-20) 지불 가능.
- **수수료(Fees) 분배**: Developer, WinnerClub, Platform 용 수수료를 자동 차감.
- **우승자 상금 배분**: 우승자에게 상금을 자동 혹은 청구 방식으로 지급.

---

## 주요 기능

1. **토너먼트 생성**:  
   - 토큰 정보(상금 토큰, 참가비 토큰), 상금액, 참가비, 최대 참가자 수, 상금 분배 방식을 지정해 새로운 토너먼트를 만듭니다.
2. **토너먼트 참가**:  
   - 참가비가 있으면 지불 후 등록, 없으면 바로 등록.
3. **참가자 리스트 셔플**:  
   - 간단한 온체인 난수(`block.prevrandao` 등)를 사용해 참가자를 무작위로 섞습니다.
4. **토너먼트 취소**:  
   - 토너먼트 생성자가 아닌 `FACTORY_ROLE` 권한을 가진 관리자가 취소할 수 있습니다. 모든 참가비는 참가자에게 환불, 상금은 생성자에게 반환.
5. **토너먼트 종료**:  
   - **수동-청구(Claim) 방식**: 우승자들은 이후 개별적으로 청구(`claimPrize`) 함수를 호출하여 상금을 수령.  
   - **자동 분배(Automatic) 방식**: 우승자가 즉시 상금을 자동으로 받음.
6. **수수료 관리**:  
   - Developer, WinnerClub, Platform 등 여러 수수료 항목을 `DEFAULT_ADMIN_ROLE` 권한으로 설정 가능.

---

## 역할(Role) 및 권한(Permissions)

스마트 컨트랙트는 `@thirdweb-dev/contracts`의 `PermissionsEnumerable`를 사용하여 역할 기반 접근 제어(RBAC)를 제공합니다.

- **DEFAULT_ADMIN_ROLE**  
  - 프로젝트의 최고 관리자 권한.  
  - 수수료 설정(Developer, WinnerClub, Platform) 및 컨트랙트 레벨 주요 설정을 변경 가능.  
- **FACTORY_ROLE**  
  - 토너먼트 운영에 필요한 주요 기능(토너먼트 종료, 취소, 셔플 등)을 실행할 수 있는 역할.  
  - 신뢰할 수 있는 주소만 이 권한을 얻도록 해야 합니다.

---

## 설치 및 설정

1. **사전 준비**  
   - [Node.js](https://nodejs.org/)  
   - [Hardhat](https://hardhat.org/) 혹은 [Truffle](https://www.trufflesuite.com/)  
   - Solidity ^0.8.0 컴파일러

## 사용 방법
- ### 토너먼트 생성
    ```solidity
    function createTournament(
        uint256 _tournamentId,
        address _prizeTokenAddress,
        address _entryTokenAddress,
        uint256 _prizeAmount,
        uint256 _entryFee,
        uint256[] memory _prizeDistribution,
        uint256 _maxParticipants
    ) external;
    ```
* 파라미터
    >* _tournamentId: 토너먼트를 구분하기 위한 고유 ID (1, 2, 100 등 중복되지 않도록).
    >* _prizeTokenAddress: 상금으로 지급할 ERC-20 토큰 컨트랙트 주소.
    >* _entryTokenAddress: 참가비로 사용할 ERC-20 토큰 컨트랙트 주소. 참가비가 0이면 실제 사용되지 않을 수 있음.
    >* _prizeAmount: 상금용으로 예치할 토큰 수량. 사전에 컨트랙트에 approve를 통해 _prizeAmount만큼 전송 허용 필요.
    >* _entryFee: 참가자가 지불해야 할 참가비. 0이면 무료 참가.
    >* _prizeDistribution: 상금 분배 비율(혹은 실제 수량)의 배열. 합이 prizeAmount와 일치해야 함.
    >* _maxParticipants: 최대 참가자 수.
* 사용예시
  ```js
    const tournamentId = 1;
    const prizeTokenAddress = "0xPrizeToken";
    const entryTokenAddress = "0xEntryToken";
    const prizeAmount = ethers.utils.parseUnits("1000", 18);
    const entryFee = ethers.utils.parseUnits("10", 18);
    const prizeDistribution = [
      ethers.utils.parseUnits("500", 18),
      ethers.utils.parseUnits("300", 18),
      ethers.utils.parseUnits("200", 18)
    ];
    const maxParticipants = 100;
    
    // 상금 토큰 컨트랙트에 approve 필요
    await prizeTokenContract.approve(managerAddress, prizeAmount);
    
    // 토너먼트 생성
    await manager.createTournament(
      tournamentId,
      prizeTokenAddress,
      entryTokenAddress,
      prizeAmount,
      entryFee,
      prizeDistribution,
      maxParticipants
    );
  ```
    
- ### 토너먼트 참가
    ```solidity
    function participateInTournament(uint256 _tournamentId) external;
  ```
    * 참가비가 있다면, 먼저 참가자 지갑에서 entryTokenAddress 토큰을 이 컨트랙트가 transferFrom 할 수 있도록 approve 해야 합니다.
    참가비가 0이면 바로 호출 가능.

- ### 참가자 제거
    ```solidity
    function removeParticipant(uint256 _tournamentId, address _participant) external onlyRole(FACTORY_ROLE);
  ```
    * FACTORY_ROLE이 있는 주소만 실행 가능.
    * 지정된 _participant를 참가 목록에서 제거하고, 참가비를 환불합니다(참가비가 있을 경우).
  
- ### 참가자 무작위 셔플
    ```solidity
    function shuffleParticipants(uint256 _tournamentId) external onlyRole(FACTORY_ROLE);
  ```
    * block.prevrandao를 이용한 온체인 난수로 참가자 배열을 무작위 섞습니다.
    * FACTORY_ROLE 권한 필요.
  
- ### 토너먼트 취소
    ```solidity
    function cancelTournament(uint256 _tournamentId) external onlyRole(FACTORY_ROLE);
    ```
    * 토너먼트를 취소하고, 모든 참가비를 참가자들에게 환불합니다.
    * 상금은 토너먼트 생성자에게 반환됩니다.
    * 해당 토너먼트는 isCancelled 플래그가 true로 설정되어 다시 활성화되지 않습니다.
  
- ### 토너먼트 종료 (수동-청구 방식)
    ```solidity
    function endTournamentC(uint256 _tournamentId, address[] memory _winners) external onlyRole(FACTORY_ROLE);
    ```
    - 상금 풀과 참가비에서 Developer, WinnerClub, Platform 수수료를 차감해 지정 주소들에 전송합니다.
    - 나머지 상금을 _winners에게 할당합니다.
    - 실제 상금은 아직 우승자 주소에 직접 전송되지 않으며, 각 우승자는 claimPrize 함수를 통해 직접 청구해야 합니다.
    - 참고
      - _winners 배열의 길이는 반드시 처음 설정한 prizeDistribution 길이와 동일해야 함.
      - 이 함수가 호출된 후, 해당 토너먼트는 더 이상 활성 상태가 아니며, 우승자들은 상금 청구를 통해 상금을 수령합니다.

- ### 토너먼트 종료 (자동 분배 방식)
    ```solidity
    function endTournamentA(uint256 _tournamentId, address[] memory _winners) external onlyRole(FACTORY_ROLE);
    ```
    - 상금 풀과 참가비에서 수수료를 차감합니다.
    - 각 우승자에게 상금을 즉시 전송합니다(자동 분배).
    - 토너먼트를 종료 상태로 설정하고 더 이상 변경 불가.
  
- ### 상금 청구 (수동-청구 방식)
    ```solidity
    function claimPrize(uint256 _tournamentId) external;
    ```
    - endTournamentC 방식을 사용하여 종료된 토너먼트의 우승자가 호출해야 합니다.
    - 자신의 우승 상금을 컨트랙트에서 받아옵니다.
    - 이미 청구했다면 다시 청구할 수 없도록 winnerPrizes[msg.sender]가 0으로 설정됩니다.

- ### 수수료 관리
  - 컨트랙트에는 세 가지 유형의 수수료가 있습니다:
    - Developer Fee
    - WinnerClub Fee
    - Platform Fee
    - 각각 0~100 사이의 퍼센트 값으로 설정됩니다. 토너먼트가 종료될 때(수동/자동) 상금 및 참가비에서 비례해 차감됩니다.

  - 수수료 설정 함수
    - DEFAULT_ADMIN_ROLE 권한으로만 설정 가능합니다.

    ```solidity
    function setDeveloperFee(address _feeAddress, uint256 _feePercent) external onlyRole(DEFAULT_ADMIN_ROLE);
    function setWinnerClubFee(address _feeAddress, uint256 _feePercent) external onlyRole(DEFAULT_ADMIN_ROLE);
    function setPlatformFee(address _feeAddress, uint256 _feePercent) external onlyRole(DEFAULT_ADMIN_ROLE);
    ```
    - _feeAddress: 해당 수수료를 받는 주소.
    - _feePercent: 100분율(%) 단위 수수료.
    
- ## 조회 함수 (View Functions)
- ### 토너먼트 정보 조회
    ```solidity
    function getTournamentInfo(uint256 _tournamentId)
        external
        view
        returns (
            address creator,
            address prizeTokenAddress,
            address entryTokenAddress,
            uint256 prizeAmount,
            uint256 entryFee,
            uint256 maxParticipants,
            bool isActive,
            bool isCancelled
        );
    ```
 - 토너먼트의 기초 정보(생성자 주소, 상금 토큰, 참가비 토큰, 상금, 참가비, 최대 참가자 수, 활성화 여부, 취소 여부)를 반환.

- ### 토너먼트 참가자 목록 조회
    ```solidity
    function getTournamentParticipants(uint256 _tournamentId) external view returns (address[] memory);
    ```
  - 참가자의 주소 배열을 반환.
  
- ### 토너먼트 상금 분배 조회
    ```solidity
    function getTournamentPrizeDistribution(uint256 _tournamentId) external view returns (uint256[] memory);
    ```
  - 토너먼트에 설정된 상금 분배 배열을 반환.
  
- ### 수수료 정보 조회
    ```solidity
    function getTournamentFees()
        external
        view
        returns (
            uint256 _developerFeePercent,
            uint256 _winnerClubFeePercent,
            uint256 _platformFeePercent,
            address _developerFeeAddress,
            address _winnerClubFeeAddress,
            address _platformFeeAddress
        );
    ```
  - 현재 설정된 수수료 퍼센트와 수수료 수령 주소를 모두 조회.

- ### 수령 가능한 상금 조회
    ```solidity
        function getClaimablePrize(uint256 _tournamentId) external view returns (uint256) {
        Tournament storage tournament = tournaments[_tournamentId];
        return tournament.winnerPrizes[msg.sender];
    }
    ```
  - 특정 토너먼트의 특정 우승자가 수령 가능한 상금을 조회.
  - 상금이 없을경우 0을 반환.