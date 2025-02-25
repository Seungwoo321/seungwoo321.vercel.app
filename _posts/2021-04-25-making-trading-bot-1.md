---
title: Amazon Athena SQL로 캔들 데이터 생성하기
date: '2021-04-25'
tags: ["Athena", "SQL", "Trading Bot"]
categories: Quant
permalink: /blog/:year/:month/:day/:title/
---

> 이 글은 2018년 4월경 테스트를 진행하면서 기록한 내용을 바탕으로 작성했습니다.

- 암호화폐 시장을 경험하면서 알고리즘 매매에 흥미가 생겼다.
- 알고리즘 매매를 위해서는 가장 기본이 되는 캔들 데이터가 필요했다.
- 어떻게 실시간 시장 데이터를 가공해서 캔들 데이터를 만들어야 하는지 흥미가 생겼다.
- AWS의 데이터 분석과 관련된 서비스 Glue와 Athena 서비스가 생각났다.

<!--more-->

## 수집 및 데이터 가공

캔들은 **종가(close)**, **고가(high)**, **저가(low)**, **시가(open)** 로 구성되어 있다.

노드(node.js)로 거래소의 API를 정기적으로 호출해서 받은 시장 데이터를 키네시스(Kinesis)로 전송하고 그다음 글루(Glue)를 사용해서 가공하고 아테나(Athena)에 최종 캔들 테이블을 생성하려고 했다.
하지만 글루 서비스를 사용하는 데서 막혔고 다른 방법을 고민하다가 아테나에서의 SQL 작성으로 원하는 시간 프레임의 캔들 데이터를 만들 수 있었다.

## SQL을 사용한 캔들 데이터 생성

구글링과 이전에 공부했었던 SQL 지식을 되뇌면서 열심히 작성한 결과 지정한 시간 프레임별로 캔들 데이터를 만드는 쿼리문을 작성할 수 있었다.

```sql
-- 5분 캔들 데이터
select
    a1.timestamp
    , a1.open_timestamp
    , b1.col1 as open
    , a1.close_timestamp
    , c1.col1 as close
    , a1.high
    , a1.low
from (
    select
        case floor(minute(from_iso8601_timestamp(col0)) / 5)
            when 0 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:00')
            when 1 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:05')
            when 2 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:10')
            when 3 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:15')
            when 4 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:20')
            when 5 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:25')
            when 6 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:30')
            when 7 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:35')
            when 8 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:40')
            when 9 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:45')
            when 10 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:50')
            when 11 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:55')
        end as timestamp
        , min(from_iso8601_timestamp(col0)) as open_timestamp
        , max(from_iso8601_timestamp(col0)) as close_timestamp
        , max(col1) as high
        , min(col1) as low
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    group by 1
    order by 1
) a1
left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) b1
on a1.open_timestamp = b1.original_timestamp
left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) c1
on a1.close_timestamp = c1.original_timestamp
where a1.timestamp is not null
order by 1
```

```sql
-- 30분 캔들 데이터
select
    a1.timestamp
    , a1.open_timestamp
    , b1.col1 as open
    , a1.close_timestamp
    , c1.col1 as close
    , a1.high
    , a1.low
from (
    select
        case floor(minute(from_iso8601_timestamp(col0)) / 30)
        when 0 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:00')
        when 1 then date_format(date_trunc('second', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:30')
        end as timestamp
        , min(from_iso8601_timestamp(col0)) as open_timestamp
        , max(from_iso8601_timestamp(col0)) as close_timestamp
        , max(col1) as high
        , min(col1) as low
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    group by 1
    order by 1
) a1
left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) b1
on a1.open_timestamp = b1.original_timestamp
left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) c1
on a1.close_timestamp = c1.original_timestamp
where a1.timestamp is not null
order by 1
```

```sql
-- 1시간 캔들 데이터
select
    a1.timestamp
    , a1.open_timestamp
    , b1.col1 as open
    , a1.close_timestamp
    , c1.col1 as close
    , a1.high
    , a1.low
from (
    select
        case floor(minute(from_iso8601_timestamp(col0)))
        when 0 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d %H:00')
        end as timestamp
        , min(from_iso8601_timestamp(col0)) as open_timestamp
        , max(from_iso8601_timestamp(col0)) as close_timestamp
        , max(col1) as high
        , min(col1) as low
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    group by 1
    order by 1
) a1 left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) b1
on a1.open_timestamp = b1.original_timestamp
left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) c1
on a1.close_timestamp = c1.original_timestamp
where a1.timestamp is not null
order by 1
```

```sql
-- 4시간 캔들 데이터
select
    a1.timestamp
    , a1.open_timestamp
    , b1.col1 as open
    , a1.close_timestamp
    , c1.col1 as close
    , a1.high
    , a1.low
from (
    select
        case floor(hour(from_iso8601_timestamp(col0)) / 4)
        when 0 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d 00:00')
        when 1 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d 04:00')
        when 2 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d 08:00')
        when 3 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d 12:00')
        when 4 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d 16:00')
        when 5 then date_format(date_trunc('hour', from_iso8601_timestamp(col0)) + interval '9' hour, '%Y-%m-%d 18:00')
        end as timestamp
        , min(from_iso8601_timestamp(col0)) as open_timestamp
        , max(from_iso8601_timestamp(col0)) as close_timestamp
        , max(col1) as high
        , min(col1) as low
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    group by 1
    order by 1
) a1 left join (
        select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
        from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) b1
on a1.open_timestamp = b1.original_timestamp
left join (
    select distinct
        from_iso8601_timestamp(col0) as original_timestamp
        , col1
    from trading.swlee_ticker
    where col2 = 'eth_krw'
    order by 1
) c1
on a1.close_timestamp = c1.original_timestamp
where a1.timestamp is not null
order by 1

```

## 실행 결과

<img src="/assets/images/posts/2021/04/25/01_sql.png" alt="write sql" />

<img src="/assets/images/posts/2021/04/25/02_result.png" alt="run sql" />

## 마무리

- 가장 기본이 되는 캔들 생성만으로 복잡하고 비용이 많이 든다.
- 글루 서비스에서 생성하는 테이블이 배치성이라서 실시간 캔들 조회가 어렵다.

원하는 시간 프레임별로 캔들 데이터를 생성하는 SQL 문을 작성하는 것은 성공했지만 위와 같은 이유들로 비효율적이라고 판단해 작업에 사용된 모든 AWS 리소스를 삭제했다.
