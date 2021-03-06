# db에 날리는 쿼리는 최대한 간단하게 하라는 의미
데이터베이스에서 할 수 있는 집계성 쿼리는 데이터베이스에서 처리하는게 좋습니다.

데이터베이스에서 데이터를 꺼낼 때는 데이터 양 자체를 줄이는 것이 매우 중요합니다.

예를 들어서 avg, sum 등은 데이터베이스가 잘 제공하는 기능이고, 또 데이터 양을 많이 줄여주기 때문에 데이터베이스에서 처리하는게 좋습니다.

그러면 어떤 경우에 데이터베이스 쿼리를 사용하는게 좋은가 하면 다음의 경우는 데이터베이스의 기능을 최대한 많이 사용해서 데이터를 줄여야 합니다.

1. 최대한 데이터를 필터링해서 가지고 오는 것(where)

2. 집계성 쿼리(avg,sum)

는 최대한 DB의 기능을 사용하는 것이 맞습니다. 그리고 집계성 쿼리도 전송하는 데이터를 크게 줄여주기 때문입니다.

유저별 랭킹 시스템을 합산하려면 유저가 100백만이라면 데이터베이스에서 avg, sum을 하면 수초만에 끝나는데, 애플리케이션에 이 데이터를 다 로딩한 다음에 집계를 내려면 수분 이상은 걸리겠지요.

그런데 제가 강조한 부분은 그 이외의 부분이라고 생각해주시면 됩니다. 예를 들어서 날짜 데이터 같은 것을 임의로 쿼리에서 조작하거나, 쿼리에서 특정 필드의 a와 b의 값을 더하고 곱하는 해서 그 결과 dto를 반환하는 행위들입니다. 위의 2가지 경우를 제외하고 데이터는 가급적 원천을 그대로 받고, 애플리케이션에서 로직으로 처리하는 것을 권장합니다.

물론 avg, sum도 데이터 양이 적거나 또는 한 페이지 안에서 처리 가능한 분량 정도면 애플리케이션에서 처리하는게 더 나은 경우도 있습니다.
