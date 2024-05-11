# 스마트 컨트랙트 함수 및 파라미터 설명서

## MiraclePenaltyPassControl 컨트랙트

### 개요
이 컨트랙트는 페널티 패스를 관리하고 구매하는 데 사용됩니다. 페널티 패스를 소유한 사용자는 일정 기간 동안 페널티를 받지 않을 수 있습니다.

### 주요함수
> #### function getPenaltyPassPrice(address tokenAddress) public view returns (uint256)
> * 설명: 지정된 토큰에 대한 페널티 패스의 가격을 반환합니다.
> * tokenAddress: address - 페널티 패스 가격을 조회할 토큰의 주소

> #### function buyPenaltyPass(address _tokenAddress) public
> * 설명: 사용자가 페널티 패스를 구매하고 소유합니다.
> * _tokenAddress: address - 구매에 사용할 토큰의 주소

> #### function hasValidPenaltyPass(address user) public view returns (bool)
> * 설명: 사용자가 유효한 페널티 패스를 가지고 있는지 확인합니다.
> * user: address - 확인할 사용자의 주소

> #### function getRemainingPenaltyPass(address user) public view returns (uint256)
> * 설명: 사용자의 남은 페널티 패스 유효 기간을 반환합니다.
> * user: address - 조회할 사용자의 주소