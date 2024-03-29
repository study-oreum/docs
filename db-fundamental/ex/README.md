# 1. 데이터베이스 일반 > 실습 관련

## 실습1

### 음료 자판기에서 콜라, 사이다, 환타를 팔아보기로 한다.

- 콜라, 사이다, 환타는 250원에 들여와서 500원에 팔기로 한다.

```sql
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (1, '콜라', 250, 500, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (2, '사이다', 250, 500, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (3, '환타', 300, 500, 350);

SELECT * FROM DRINK;
```

### 흰우유, 야쿠르트를 추가로 팔아보기로 한다.
- 흰우유는 100원에 들여와서 300원에 팔기로 한다. 야쿠르트는 20원에 들여와서 400%의 이윤을 남기기로 한다.
```sql
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (4, '흰우유', 100, 300, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (5, '야쿠르트', 20, 100, 350);

SELECT * FROM DRINK;
```

## 실습2

### 환타의 원가가 50원 올랐다.

```sql
UPDATE DRINK SET BUY = BUY + 50
WHERE ID = 3;

SELECT * FROM DRINK
WHERE ID = 3;
```

### 웰치스를 추가로 팔아보기로 한다. 근데 사이즈가 500ml이다.

- 웰치스는 400원에 들여와서 1,000원에 팔기로 한다.

```sql
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (6, '웰치스', 400, 1000, 500);

SELECT * FROM DRINK;
```

## 실습3

```sql
-- 1번 문제 - 이윤이 가장 높은 음료는?
-------------------------------
SELECT NAME, (SELL - BUY) AS INCOME FROM DRINK ORDER BY INCOME DESC LIMIT 1;
-- 웰치스 / 600

-- 2번 문제 - 용량당 단가가 가장 낮은 음료?
-----------------------------------
SELECT NAME, (SELL / MILLILITER) AS ML_PER_PRICE FROM DRINK ORDER BY ML_PER_PRICE DESC LIMIT 1;
-- 웰치스 / 2

-- 3번 문제 - 현재(2019-9-30) 각 음료별 재고 수량은?
--------------------------------------------
SELECT
	D.NAME, TT.CNT
FROM (
	SELECT
		T.DRINK_ID, COUNT(T.CNT) AS CNT
	FROM (
		SELECT DRINK_ID, CNT FROM BUY
		UNION
		SELECT DRINK_ID, CNT FROM SELL
	) T
	GROUP BY DRINK_ID
) TT LEFT OUTER JOIN DRINK D ON TT.DRINK_ID = D.ID;
-- 위에서 잘못된 부분을 수정해보자
SELECT
	D.NAME, TT.CNT
FROM (
	SELECT
		T.DRINK_ID, SUM(T.CNT) AS CNT -- 음료별 수량을 합산
	FROM (
		SELECT DRINK_ID, (CNT*-1) AS CNT FROM BUY -- 판매분을 음수로 변환
		UNION
		SELECT DRINK_ID, CNT FROM SELL
	) T
	GROUP BY DRINK_ID
) TT LEFT OUTER JOIN DRINK D ON TT.DRINK_ID = D.ID;

-- 4번 문제 - 현재(2019-9-30) '총매출'과 '순이익'은?
---------------------------------------------
-- 총매출
SELECT
	SUM(B.CNT * D.SELL)
FROM SELL B LEFT OUTER JOIN DRINK D ON B.DRINK_ID = D.ID;
-- 4000

-- 순이익
SELECT
	SUM(B.CNT * (D.SELL - D.BUY))
FROM SELL B LEFT OUTER JOIN DRINK D ON B.DRINK_ID = D.ID;
-- 2050

-- 5번 문제 - 2019-1-1 콜라가 판매된 총수량은?
---------------------------------------
SELECT
	SUM(CNT)
FROM SELL
WHERE DRINK_ID = 1
AND REG_DATE = '2019-01-01';
-- 3
```

## 실습4

### 판매번호 7번에 대한 고객 환불이 필요하다.

```sql
DELETE FROM SELL
WHERE SEQ = 7;

SELECT * FROM SELL
WHERE SEQ = 7;
```

________________________________________________________________________________


## 실습을 위한 테이블 생성
```sql
-- oreum study > 데이터베이스 일반 > 실습 SQL
DROP TABLE DRINK;
DROP TABLE SELL;
DROP TABLE BUY;

-- 음료 테이블
CREATE TABLE IF NOT EXISTS `DRINK` (
  `ID` INT NOT NULL COMMENT '고유번호',
  `NAME` VARCHAR(255) NOT NULL COMMENT '이름',
  `BUY` INT NOT NULL COMMENT '원가',
  `SELL` INT NOT NULL COMMENT '판매가',
  `MILLILITER` INT NOT NULL COMMENT '용량(ml)',
  PRIMARY KEY (`ID`))
ENGINE = InnoDB
COMMENT = '음료';

-- 판매 테이블
CREATE TABLE IF NOT EXISTS `SELL` (
  `SEQ` INT NOT NULL,
  `DRINK_ID` INT NOT NULL COMMENT '음료 고유번호',
  `CNT` INT NOT NULL COMMENT '수량',
  `REG_DATE` DATETIME NOT NULL COMMENT '판매일',
  PRIMARY KEY (`SEQ`),
  INDEX `fk_SELL_DRINK1_idx` (`DRINK_ID` ASC),
  CONSTRAINT `fk_SELL_DRINK1`
    FOREIGN KEY (`DRINK_ID`)
    REFERENCES `DRINK` (`ID`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB
COMMENT = '판매';

-- 입고 테이블
CREATE TABLE IF NOT EXISTS `BUY` (
  `ID` INT NOT NULL COMMENT '입고번호',
  `DRINK_ID` INT NOT NULL COMMENT '음료 고유번호',
  `CNT` INT NOT NULL COMMENT '수량',
  `REG_DATE` DATETIME NOT NULL COMMENT '입고일',
  PRIMARY KEY (`ID`, `DRINK_ID`),
  INDEX `fk_BUY_DRINK_idx` (`DRINK_ID` ASC),
  CONSTRAINT `fk_BUY_DRINK`
    FOREIGN KEY (`DRINK_ID`)
    REFERENCES `DRINK` (`ID`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;
```

## 최종 결과 데이터

```sql
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (1, '콜라', 250, 500, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (2, '사이다', 250, 500, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (3, '환타', 300, 500, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (4, '흰우유', 100, 300, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (5, '야쿠르트', 20, 100, 350);
INSERT INTO `DRINK` (`ID`, `NAME`, `BUY`, `SELL`, `MILLILITER`) VALUES (6, '웰치스', 400, 1000, 500);
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (1, 1, 50, '2019-01-01 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (2, 2, 50, '2019-01-01 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (3, 3, 50, '2019-01-01 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (4, 4, 10, '2019-03-01 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (5, 5, 10, '2019-03-01 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (6, 6, 30, '2019-05-05 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (7, 4, 10, '2019-06-25 00:00:00');
INSERT INTO `SELL` (`SEQ`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (8, 1, 10, '2019-08-15 00:00:00');
INSERT INTO `BUY` (`ID`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (1, 1, 3, "2019-01-01 00:00:00");
INSERT INTO `BUY` (`ID`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (2, 1, 1, "2019-01-02 00:00:00");
INSERT INTO `BUY` (`ID`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (2, 2, 1, "2019-01-02 00:00:00");
INSERT INTO `BUY` (`ID`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (2, 3, 1, "2019-01-02 00:00:00");
INSERT INTO `BUY` (`ID`, `DRINK_ID`, `CNT`, `REG_DATE`) VALUES (2, 6, 1, "2019-01-02 00:00:00");
```
