
<카드사 은행사 요청 자료 추출>

1.

프로젝트명: 신규 사업 추진을 위한 샘플 카드 상품의 조건 별 매출 건수 및 매출액 산출

작성시점: 2022.04.04~2022.04.05(2영업일)

핵심목표: 생애 최초 카드 발급한 연령대 비율 활용하여 사업 추진

요청대상: 고객사팀

요청내용: 1) 최근 5년 기준 특정 제휴 회원사에 대한 연도별 카드 상품 및 연령대 별 비율 및 회원 수 산출(10~60대 이상),

            등록 년도(최근 5년) | 연령대(10~60대) | 회원 수 | 등록 년도 및 연령대 기준 백분율

 

            2) 최근 5년 기준 전체 전체 회원사에 대한 개인고객 대상 연도별 연령대 별 비율 및 회원 수 산출

            카드상품번호 | 등록 년도(최근 5년) | 연령대(10~60대) | 회원 수 | 등록 년도 및 연령대 기준 백분율

 

해결방법: 카드 등록일과 생년월일간 months_between 하여 12개월로 나누었으며 trunc으로 소수점을 절사하여 연령 산출함

Partition by를 Product_no, year 기준으로 나누어 cust 건수를 산출한 뒤 백분율 구함 소수점 뒷자리 1~2자리를 제외 하고 반올림

활용프로그램: PL/SQL, EXCEL

 

관련 쿼리

1)     최근 5년 기준 특정 제휴사에 대한 연도별 카드 상품 및 연령대 별 비율 및 회원 수 산출

 

Create table hk_special_2204 as

Select

Year,

연령대,

cnt,

round(ratio_to_report(cnt) over (partition by Product_no,year),2)*100 || ‘%’ as ratio_cnt

-- Product_no,year 기준으로 cust 백분율 구함(뒷자리 2자리)로 하고 반올림

from(

Select

/*+full(a) parallel(a 8) full(b) parallel(b 8)*/

?  FULL PARALLEL은 데이터의 용량이 큰 나눠서 빠르게 실행하여 결과값을 산출하게 하는 역할을 한다.이를 튜닝이라고 한다. PARALLEL은 최대 36으로 할당하는 것이 좋다. 괄호()에 값을 넣는 것을 힌트라고 지칭한다.

Product_no,

Substr(date,1,4)year,

Case when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0)<=’18’ Then ‘10대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’19’ AND ’28’ Then ‘20대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’29’ AND ’38’ Then ‘30대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’39’ AND ’48’ Then ‘40대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’49’ AND ’58’ Then ‘50대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) <=’59’  Then ‘60대 이상’ end “연령대”

--문자열 형식변환 text->date month_between 을 통해 기간 차이 산출 후 나누기 12개월을 함으로서 연령 산출 case when 조건으로 연령대 나누기(만나이 기준 산출)

10/60대 이상 제외 BETWEEN 조건으로 연령대 산출

, COUNT(DISTINCT A.CUST_NO) CNT

FROM CA_PRODUCT_TABLE A,

( SELECT

/*+full(a) parallel(a 8) full(b) parallel(b 8)*/

CA_CUST_NO,

CA_HWON_NO,

AGE,

IDENTITY_NO_S,

CASE WHEN SUBSTR(IDENTITY_NO_S,1,2)BETWEEN ‘00’ AND ‘22’ AND AGE <=’30’ THEN ‘20’|| IDENTITY_NO ELSE ‘19’|| IDENTITY_NO END ID_NO

-- IDENTITY_NO를 SUBSTR로 앞 2자리 활용 앞에 00~22 이면서 AGE가 30 보다 작거나 같을 경우 IDENTITY_NO 앞에 20을 붙이고 그렇지 않은 경우

-- IDENTITY_NO 앞에 19를 붙인다. EX) BORN_DATE(YYYY.MM)산출

FROM ID_TABLE

WHERE individual_ corporate_CD= ‘1’

AND BUSINEE_NO=’000’

)B

WHERE  issue_cd in (‘00’,’01’)

--신규(여기서 아예 최초), 추가신규(기존에 있지만 이거는 최초)

And ca_product_no in (‘0000’,’0001’)

And date between ‘2017%’ and ‘2022%’

Group by Product_no, Substr(date,1,4)year,

Case when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0)<=’18’ Then ‘10대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’19’ AND ’28’ Then ‘20대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’29’ AND ’38’ Then ‘30대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’39’ AND ’48’ Then ‘40대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’49’ AND ’58’ Then ‘50대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) <=’59’  Then ‘60대 이상’ end

) group by Product_no, year, 연령대, cnt

Order by 1,2,3,4 desc --(내림차순)

 

2) 최근 5년 기준 전체 고객사에 대한 개인고객 대상 연도별 연령대 별 비율 및 회원수 산출

 

Create table hk_special_2204 as

Select

Year,

연령대,

cnt,

round(ratio_to_report(cnt) over (partition by year),1)*100 || ‘%’ as ratio_cnt

-- year 기준으로 cust 백분율 구함(뒷자리 1자리)로 하고 반올림

from(

Select

/*+full(a) parallel(a 8) full(b) parallel(b 8)*/

Substr(date,1,4)year,

Case when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0)<=’18’ Then ‘10대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’19’ AND ’28’ Then ‘20대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’29’ AND ’38’ Then ‘30대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’39’ AND ’48’ Then ‘40대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’49’ AND ’58’ Then ‘50대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) <=’59’  Then ‘60대 이상’ end “연령대”

--문자열 형식변환 text->date month_between 을 통해 기간 차이 산출 후 나누기 12개월을 함으로서 연령 산출 case when 조건으로 연령대 나누기(만나이 기준 산출)

10/60대 이상 제외 BETWEEN 조건으로 연령대 산출

, COUNT(DISTINCT A.CUST_NO) CNT

FROM CA_PRODUCT_TABLE A,

( SELECT

/*+full(a) parallel(a 8) full(b) parallel(b 8)*/

CA_CUST_NO,

CA_HWON_NO,

AGE,

IDENTITY_NO_S,

CASE WHEN SUBSTR(IDENTITY_NO_S,1,2)BETWEEN ‘00’ AND ‘22’ AND AGE <=’30’ THEN ‘20’|| IDENTITY_NO ELSE ‘19’|| IDENTITY_NO END ID_NO

-- IDENTITY_NO를 SUBSTR로 앞 2자리 활용 앞에 00~22 이면서 AGE가 30 보다 작거나 같을 경우 IDENTITY_NO 앞에 20을 붙이고 그렇지 않은 경우

-- IDENTITY_NO 앞에 19를 붙인다. EX) BORN_DATE(YYYY.MM)산출

FROM ID_TABLE

WHERE individual_ corporate_CD= 1

AND BUSINEE_NO=’000’

)B

WHERE  issue_cd in (‘00’,’01’)

--신규(여기서 아예 최초), 추가신규(기존에 있지만 이거는 최초)

And date between ‘2017%’ and ‘2022%’

Group by Substr(date,1,4)year,

Case when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0)<=’18’ Then ‘10대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’19’ AND ’28’ Then ‘20대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’29’ AND ’38’ Then ‘30대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’39’ AND ’48’ Then ‘40대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’49’ AND ’58’ Then ‘50대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) <=’59’  Then ‘60대 이상’ end

) group by year, 연령대, cnt

Order by 1,2,3 desc --(내림차순)

 

2.

프로젝트명: 광주광역시, 목포시 유효 가맹점 명세 산출

작성시점: 2022.04.12~2022.04.13(2영업일)

핵심목표: 광주, 목포 지역화폐 캐시백 대상 가맹점 최신화(update)

[1] 국세청 전통시장 가맹점과 광주/목포 지역 가맹점 매핑(해당 지역 전통시장 가맹점 산출)

[2] 국세청 전통시장 가맹점 자료와 비교

[3] 그룹가맹점 신규, 해지 대상 가맹점 추출 및 서버 반영

 

요청대상: 광주지점(비씨)

요청내용: 가맹점명 | 가맹점명 | 업종코드 | 업종명 | 주소(지역+시.군.구) | 가맹점등급

 

<관련쿼리>

 

Select

/*+ full(a) paralle(a 8) full(b) paralle(b 8) full(c) paralle(c 8) compress nologging(a+b+c) */

D.지역,

A. 가맹점번호,

A. 가맹점명,

A. 업종코드,

B.업종명,

C. 지역 || C. 시,군.구 AS 주소,

DECODE(가맹점등급코드, T, ’중소’, ‘3’,영세’,’일반’) 가맹점등급,

* DECODE란 어떤 컬럼의 값에 대해 조건을 걸어 그에 해당하는 경우 별명을 정해주고 이도저도 해당하지 않는 값은 기타의 별명을 주는 함수이다.

FROM 가맹점 Information A, 가맹점/업종code B, 주소 data C ,

( Select

Distinct Decode( substr(b.지역명,1,2), ‘광주’,’광주’,’목포’) 지역,

A.우편번호

From zp번호 A, 지역DATA B

Where substr(a.우편번호,1,3)=b.우편지역번호

And b.우편번호유형코드=’02’ --신주소

And substr(b.지역명,1,5) in (‘광주 북구’,’광주 남구’ ,’광주 광산구’ ,’광주 서구’ ,’전남 목포’ ,’광주 동구’)

) d

Where a.거래상태코드 < ‘4’ (1:가능, 2:경고, 3:정지,4:임의해지,5:해지, 9:기타)

And nvl(a.해지일자,’99991231’) > ‘20220531’

And a.가맹점번호=b.가맹점번호(+)

And a.가맹점업종번호= b.가맹점업종번호(+)

And a.사업장우편번호= b.사업장우편번호 ;

 

3.

프로젝트명: 네이버파이낸셜 신규 사업 추진을 위한 샘플 카드상품의 조건별 매출건수 및 매출액 산출

작성시점: 2022.04.15~2022.04.25(7영업일)

핵심목표: 특정 카드상품의 그룹 가맹점별 매출현황을 활용하여 신규카드 사업 추진.

요청대상: 고객사팀

요청내용: 개인 회원의 21’년 하반기 연령대 및 가맹점별 매출 산출

카드상품번호 | 카드고객번호 | 대체카드번호 | 등록년월 | 연령대(10~60대 이상) | 전월실적() | 총 매출 | 온라인 매출 | 오프라인 매출 | 네이버페이 QR매출 | 네이버페이멤버십 매출 |

오프라인 취미 매출 | 항공사/면세점 매출

산출방식:

등록일자와 생년월일 간 month count하고 12개월로 나누어 연령대 계산

 

 

<관련쿼리>

 

<previous month performance-1>

Create table PMP_0425 AS

SELECT

/*+FULL(A) PARALLEL(A 16)*/

Card_pdct_no,

Card_cust_no,

Altn_card_no,

Reg_yymm,--등록일자

Sum (amt_all) amt_all

FROM day_card_seles

WHERE cumuse_sale_ctgo_cd < 4 (자사매출)

And reg_yymm between ‘202106’ and ‘202111’ (등록일자)

And 가맹점_category_code not in (‘2356’, ’9756’)정산 가맹점 제외

And card_cust_corporate_or_individual= ‘1’(본인)

And sale_ctgegory_cd not like ‘%7’?특정서비스제외

And card_pdct_no =’123456’

Group by Card_pdct_no, Card_cust_no, Altn_card_no, Reg_yymm

 

Union all

 

SELECT

/*+FULL(A) PARALLEL(A 16)*/

Card_pdct_no,

Card_cust_no,

Altn_card_no,

substr(rcv_date,1,6)Reg_yymm,

sum(case when ) amt_all

FROM foreign currency a,

WHERE rcv_date between ‘20210601’ and ‘20211130’

AND card_pdct_no =’123456’

AND sale_ctgegory_cd not like ‘%7’?특정서비스제외

GROUP BY Card_pdct_no, Card_cust_no, Altn_card_no, substr(rcv_date,1,6)

) a

Group by Card_pdct_no, Card_cust_no, Altn_card_no, Reg_yymm

 

<previous month performance-2>

 

Create table PMP_0425_1 as

Select

Card_pdct_no,

Card_cust_no,

Altn_card_no,

Reg_yymm as 전월,

amt_all,

case when reg_yymm =’202106’ and ‘202107’

     when reg_yymm =’202107’ and ‘202108’

     when reg_yymm =’202108’ and ‘202109’

     when reg_yymm =’202109’ and ‘202110’

     when reg_yymm =’202110’ and ‘202111’

     when reg_yymm =’202111’ and ‘202112’

end reg_yymm - -전체실적 산출 시 매칭값을 위해 임의 변환

from PMP_0425 ;

 

 

< Merchants by category -1 on/off line pay>

 

 Create table Mer_cate_0424 as

Select

Card_pdct_no,

Card_cust_no,

Altn_card_no,--대체번호

Reg_yymm,--등록일자

Sum (amt_all) amt_all,

Sum (cnt_all) cnt_all,

Sum (amt_on) amt_on,

Sum (cnt_on) cnt_on,

Sum (amt_off) amt_off,

Sum (cnt_off) cnt_off

FROM (

SELECT

/*+full(a) parallel(a 8) full(b) parallel(b 8)*/

a.Card_pdct_no,

a.Card_cust_no,

a.Altn_card_no,--대체번호

a.Reg_yymm,--등록일자

NVL( Sum(amt_all),0) amt_all,

NVL( Sum (case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END) ,0) cnt_all,

NVL(Sum ( case when A.MER_TPBUZ_NO IN (‘9720’,’9302’) THEN A.SALE_AMT END) ,0 ) amt_on,

NVL(Sum ( case when A.MER_TPBUZ_NO IN (‘9720’,’9302’) AND SALE_GUBUN_CD LIKE ‘0%’ THEN 1

When A.MER_TPBUZ_NO IN (‘9720’,’9302’) AND SALE_GUBUN_CD LIKE ‘1%’ THEN 1 ELSE 0 END) ,0 ) cnt_on

FROM day_card_seles a

WHERE a.cumuse_sale_ctgo_cd < 4 (자사매출)

And a.reg_yymm between ‘202106’ and ‘202111’ (등록일자)

And a.가맹점_category_code not in (‘2356’, ’9756’)정산 가맹점 제외

And a.card_cust_corporate_or_individual= ‘1’(본인)

And a.sale_ctgegory_cd not like ‘%7’?특정서비스제외

And a.card_pdct_no =’123456’

Group by a.Card_pdct_no, a.Card_cust_no, a.Altn_card_no, a.Reg_yymm

 

Union all

 

SELECT

/*+FULL(A) PARALLEL(A 16)*/

Card_pdct_no,

Card_cust_no,

Altn_card_no,

substr(rcv_date,1,6)Reg_yymm,

sum(case when cancle_sale_yn=’Y’ THEN -1*SALE_WON_AMT ELSE SALE_WON_AMT END ) amt_all,

sum(case when cancle_sale_yn=’Y’ THEN -1 ELSE 1) cnt_all,

0 amt_on,

0 cnt_on

FROM foreign currency a,

WHERE rcv_date between ‘20210601’ and ‘20211130’

AND card_pdct_no =’123456’

AND sale_ctgegory_cd not like ‘%7’?특정서비스제외

GROUP BY Card_pdct_no, Card_cust_no, Altn_card_no, substr(rcv_date,1,6)

) a

Group by Card_pdct_no, Card_cust_no, Altn_card_no, Reg_yymm

;

 

< Merchants by category -2 QR pay>

 

 Create table Mer_cate_0424_1 as

Select

/*+full(a) parallel(a 8) */

Card_pdct_no,

Card_cust_no,

Altn_card_no,

Reg_yymm,

Sum(amt_all) amt_all,

NVL( Sum (case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END) ,0) cnt_all

From day_card_seles a

WHERE a.cumuse_sale_ctgo_cd < 4 (자사매출)

And a.reg_yymm between ‘202106’ and ‘202111’ (등록일자)

And a.token_naver_no=’1234567890’

And a.internet_qr_pay_code=’13’

And a.가맹점_category_code not in (‘2356’, ’9756’)정산 가맹점 제외

And a.card_cust_corporate_or_individual= ‘1’(본인)

And a.sale_ctgegory_cd not like ‘%7’?특정서비스제외

And a.card_pdct_no =’123456’

Group by a.Card_pdct_no, a.Card_cust_no, a.Altn_card_no, a.Reg_yymm ;

 

< Merchants by category -3 Off line hobby>

 

Create table Mer_cate_0424_2 as

Select

Card_pdct_no,

Card_cust_no,

Altn_card_no,

Reg_yymm,

NVL(SUM(CASE WHEN a.sbmall_code in (‘123456789’,’4567891234’,’987654321’) THEN

CASE WHEN sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end),0) CNT1,

NVL(SUM( case when a.sbmall_code in (‘123456789’,’4567891234’,’987654321’) THEN sale_amt

end),0)AMT

From day_card_seles a

WHERE a.cumuse_sale_ctgo_cd < 4 (자사매출)

And a.reg_yymm between ‘202106’ and ‘202111’ (등록일자)

And a.card_cust_corporate_or_individual= ‘1’(본인)

And a.sale_ctgegory_cd not like ‘%7’?특정서비스제외

And a.card_pdct_no =’123456’

Group by a.Card_pdct_no, a.Card_cust_no, a.Altn_card_no, a.Reg_yymm ;

 

<ALL TABLE >

 

Create table Mer_cate_0424_all_1 as

Select

a.Card_pdct_no,

a.Card_cust_no,

a.Altn_card_no,

a.Reg_yymm,

nvl(b.amt_all,0) previous_month_sale,

a.cnt_all all_sales_cnt,

a.amt_all all_sales_amt,

a.cnt_on all_sales_cnt_on,

a.amt_on all_sales_amt_on,

a.cnt_off all_sales_cnt_off,

a.amt_off all_sales_amt_off,

nvl(c.cnt_all ) all_sales_cnt_QR,

nvl(c.amt_all) all_sales_amt_QR,

nvl(e.cnt_all) all_sales_cnt_off_line_hobby,

nvl(e.amt_all) all_sales_amt_off_line_hobby

from Mer_cate_0424 a,--on/off

PMP_0425_1 b,-- previous month

Mer_cate_0424_1 c, -?qr

Mer_cate_0424_2 d--off line hobby

Where a. Altn_card_no = b. Altn_card_no(+)

And a.reg_yymm=b.reg_yymm(+)

And a. Altn_card_no = c. Altn_card_no(+)

And a.reg_yymm=c.reg_yymm(+)

And a. Altn_card_no = d. Altn_card_no(+)

And a.reg_yymm=d.reg_yymm(+)

Group by a.Card_pdct_no, a.Card_cust_no, a.Altn_card_no, a.Reg_yymm

 

 

<ALL TABLE_age >

Create table Mer_cate_0424_all_1 as

Select

a.Card_pdct_no,

a.Card_cust_no,

a.Altn_card_no,

a.Reg_yymm,

Case when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0)<=’18’ Then ‘10대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’19’ AND ’28’ Then ‘20대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’29’ AND ’38’ Then ‘30대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’39’ AND ’48’ Then ‘40대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’49’ AND ’58’ Then ‘50대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) <=’59’  Then ‘60대 이상’ end “연령대”,

a.previous_month_sale,

a.all_sales_cnt,

a.all_sales_amt,

a.all_sales_cnt_on,

a.all_sales_amt_on,

a.all_sales_cnt_off,

a.all_sales_amt_off,

a.all_sales_cnt_QR,

a.all_sales_amt_QR,

a.all_sales_cnt_off_line_hobby,

a.all_sales_amt_off_line_hobby

from Mer_cate_0424_all_1 a, (

Select

/*+full(a) parallel(16) */

Card_cust_no,

AGE,

IDENTITY_NO_S,

CASE WHEN SUBSTR(IDENTITY_NO_S,1,2)BETWEEN ‘00’ AND ‘22’ AND AGE <=’30’ THEN ‘20’|| IDENTITY_NO ELSE ‘19’|| IDENTITY_NO END ID_NO

FROM ID_TABLE

WHERE individual_ corporate_CD= 1

AND BUSINEE_NO=’000’

)B

Where a. Card_cust_no = b.Card_cust_no(+)

Group by a.Card_pdct_no,

a.Card_cust_no,

a.Altn_card_no,

a.Reg_yymm,

Case when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0)<=’18’ Then ‘10대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’19’ AND ’28’ Then ‘20대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’29’ AND ’38’ Then ‘30대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’39’ AND ’48’ Then ‘40대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) BETWEEN ’49’ AND ’58’ Then ‘50대’

when trunc(months_between (to_date(A.date,1,6),’yyyymm’),to_date(substr(id_no,1,6_,’yyyymm’))/12,0) <=’59’  Then ‘60대 이상’ end ,

a.previous_month_sale,

a.all_sales_cnt,

a.all_sales_amt,

a.all_sales_cnt_on,

a.all_sales_amt_on,

a.all_sales_cnt_off,

a.all_sales_amt_off,

a.all_sales_cnt_QR,

a.all_sales_amt_QR,

a.all_sales_cnt_off_line_hobby,

a.all_sales_amt_off_line_hobby

4.

프로젝트명: ibk 페이북 온/오프라인 결제 내역 산출

작성시점: 2022.04.25(1영업일)

핵심목표: 자사의 페이북결제(오프라인)/qr결제(온라인) 비율 및 매출액 현황을 활용하여 마케팅 타깃 설정

요청대상: 회원사팀

요청내용: 자사/ibk의 최근 4개년 페이북 온/온프라인 매출건수 및 매출액

 

<관련쿼리>

1) ibk

Create table hk_pay_ibk as

Select

/*+ full(a) parallel(a 16) */

substr(a.reg_yymm,1,4) as “REG_YYYY

,SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code in (‘20’,’97’) -? 온라인 구분

then case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end) pay_on_cnt

 

, SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code in (‘20’,’97’) -? 온라인 구분

then sale_amt end) pay_on_amt

 

,SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code not in (‘20’,’97’) -? 온라인 구분

then case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end) pay_off_cnt

 

, SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code not in (‘20’,’97’) -? 온라인 구분

then sale_amt end) pay_off_amt

 

From day_card_seles a

Where a.reg_yymm between ‘201601’ and ‘202112’

And a. member_company_code=’001’--ibk

And cumuse_sale_ctgo_cd < 4 (자사매출)

Group by substr(a.reg_yymm,1,4)

Order by 1 desc ;

 

2) 자사

Create table hk_pay_bc as

Select

/*+ full(a) parallel(a 16) */

substr(a.reg_yymm,1,4) as “REG_YYYY

,SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code in (‘20’,’97’) -? 온라인 구분

then case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end) pay_on_cnt

 

, SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code in (‘20’,’97’) -? 온라인 구분

then sale_amt end) pay_on_amt

 

,SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code not in (‘20’,’97’) -? 온라인 구분

then case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end) pay_off_cnt

 

, SUM(case when (( a.token_naver_no=’1234567890’) or (token_naver_no=’1234567891’)

Or (electronic_commerce_datail_code in ‘1’, ‘2’)?공인인증결제, 모바일 결제

or (electronic_commerce_datail_code is null)

and electronic_commerce_type_code=’1’))?모바일 결제

and online_type_code not in (‘20’,’97’) -? 온라인 구분

then sale_amt end) pay_off_amt

From day_card_seles a

Where a.reg_yymm between ‘201601’ and ‘202112’

And cumuse_sale_ctgo_cd < 4 (자사매출)

Group by substr(a.reg_yymm,1,4)

Order by 1 desc ;

 

5.

프로젝트명: 정 회원사 최근 출시 카드상품 별 유효카드 수 산출

작성시점: 2022.04.26~2022.04.27(2영업일)

핵심목표:

1)    11개 회원사의 각 회원사별 최근 출시된 신용카드 카드상품 확인하여 판매 실적 포인트 활용목적

2)    고객사 상품발급 활성화를 위한 판매 제안 기초자료용으로 활용

3)    해당 카드상품별 총 발급좌수 및 현재 기준 유효카드좌수 확인 목적

요청대상: 회원사팀

요청내용: 2010.01~2022.03까지의 기간의 11개 회원사 신용카드 유효건수 및 전체 건수 추출

회원사명| 회원사번호 | 카드상품명 | 카드상품코드 | 개인/기업 구분 | 카드 출시월 | 총 카드발급수 |

유효카드수

 

<관련쿼리>

Create table hk_card_use_cnt as

Select

/*+ full(a) parallel(a 8) full(b) parallel(b 8) compress nologing (a+b+c)*/

Decode(a.member_company_code, ‘002’,’001’, a.member_company_code) member_company_code ,

?002를 001로 변경처리

Decode(a.member_company_nm, ‘농축협’,’농협’, a.member_company_nm) member_company_nm,

? 농축협을 농협으로 변경처리

c.Card_pdct_nm,

a.Card_pdct_no,

case when a.card_possess_type_code in (‘1’,’2’) then ‘개인’ ‘법인’ end individual_ corporate_CD,

substr(c.reg_date,1,6) reg_yymm,

sum(a.use_card_num) use_card ?유효카드 수

From month_Client_company_card_total a, Client_company_code b, card_pdct_basic c

Where a. member_company_code = b. member_company_code

And a.Card_pdct_no =c. a.Card_pdct_no(+)

And a.Client_company_type_code=’1’--정회원사

And a.standard_date=’202203’ ? 3월 기준의 유효건수

And substr(c.reg_date,1,6) between ‘201001’ and ‘202203’

And a.Client company_type_code=’3’?신용카드

Group by a.member_company_code, a.member_company_nm, c.Card_pdct_nm, a.Card_pdct_no,

case when a.card_possess_type_code in (‘1’,’2’) then ‘개인’ ‘법인’ end, substr(c.reg_date,1,6) r

Order by Decode(a.member_company_nm, ‘농축협’,’농협’, a.member_company_nm),

substr(c.reg_date,1,6), c.Card_pdct_nm

 

6.

프로젝트명: 카카오페이 매출금액 구간별 월별 매출현황 산출

작성시점: 2022.05.12~2022.05.18(5영업일)

핵심목표: 카카오페이 금융감독원 정기보고 자료 작성 목적

요청대상: 고객사팀

산출기준:

- 월별 매출액 및 매출건수 추출 -> power 함수(제곱근)를 통해 매출액 금액 단위 별 그룹 분류

           

요청내용:

1) 22년 1~3월 기간의 가맹점등급 및 금액 구간별 월별 매출 현황

등록 월, (가맹점 유형구분) 3억이하(영세),10억이하(소기업),100억이하(중소),500억이하(중견),500억 초과 (대기업),매출금액, 승인금액, 매출건수

 

2) 22년 1~3월 기간의 금액 구간별 월별 온.오프라인 매출 현황

등록 월, (매출액 유형구분) 3억이하,10억이하,100억이하,500억이하, 500억 초과, 온.오프라인매출액, 온.오프라인 매출건수

 

<관련쿼리>

1)    가맹점등급 및 금액 구간별 월별 매출 현황

등록 월, (가맹점 유형구분) 3억이하(영세),10억이하(소기업),100억이하(중소),500억이하(중견),500억 초과 (대기업),매출금액, 승인금액, 매출건수


1-1 금액 구간별 매출액 및 가맹점번호 산출

 

Create table hk_pay_cate_2205 as

Select

/*+full(a) parallel (a 8) */

Mer_no,?가맹점번호

Case when amt<=porwer (10,8)*3 then ‘3억이하’

?10*10*10*10*10*10*10*10 (10의8 제곱근)

when amt<=porwer (10,9)*1 then ‘10억이하’

when amt<=porwer (10,10)*1 then ‘100억이하’

when amt<=porwer (10,10)*5 then ‘500억이하’

when amt>porwer (10,10)*5 then ‘500억초과’

END GUBUN ,

TO_CHAR (AMT,’fm999,999,999,999,999’)amt01 ?max 기준치로 금액단위 구분(,) 나누기

From (

Select

Mer_no,--가맹점번호

Sum(일시불매입금액+할부매입금액+즉시불매입금액+선불카드매입금액)amt

From month_12_mer_total

Where standard_date=’202203’

Group by mer_no

)

;

 

1-2 가맹점등급별 금액단위 별 승인/매출액 및 매출건수 산출

 

Create table hk_pay_cate_amt_2205 as

Select

a.*

from (

select

/*+full(a) parallel (a 8) */

Case when b.gubun= ‘3억이하’ and c.mer_grd_cd=’1’ then ‘영세기업’

? mer_grd_cd(가맹점 등급)

when b.gubun= ‘10억이하’ and c.mer_grd_cd=’2’ then ‘소기업’

when b.gubun= ‘100억이하’ and c.mer_grd_cd=’3’ then ‘중소기업’

when b.gubun= ‘500억이하’ and c.mer_grd_cd=’4’ then ‘중견기업’

when b.gubun= ‘500억초과’ and c.mer_grd_cd=’5’ then ‘대기업’

END ‘mer_type’ ,

A, reg_yymm, ?등록월

,sum(sale_amt) amt_1 ?매출금액

,sum(auth_amt) amt_2?카드승인금액

,Sum(case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end) cnt ?매출코드가 정상인것만

From day_card_seles a, hk_pay_cate_2205 b, day_mer_resarch c

Where a.mer_no=b.mer_no

And a.mer_no=c,mer_no

And a.reg_yymm between ‘202201’ and ‘202203’

And a.mb_no=’008’?제휴사코드

And cumuse_sale_ctgo_cd < 4 (자사매출)

Group by Case when b.gubun= ‘3억이하’ and c.mer_grd_cd=’1’ then ‘영세기업’

when b.gubun= ‘10억이하’ and c.mer_grd_cd=’2’ then ‘소기업’

when b.gubun= ‘100억이하’ and c.mer_grd_cd=’3’ then ‘중소기업’

when b.gubun= ‘500억이하’ and c.mer_grd_cd=’4’ then ‘중견기업’

when b.gubun= ‘500억초과’ and c.mer_grd_cd=’5’ then ‘대기업’ END,

A. reg_yymm

)a

Where mer_type is not null

Order by 1,2 asc;

 

2 금액 구간별 월별 온.오프라인 매출 현황

 

1-1 금액 구간별 매출액 및 가맹점번호 산출

 

Create table hk_pay_cate_on_off_2205 as

Select

/*+full(a) parallel (a 8) */

Mer_no,?가맹점번호

Case when amt<=porwer (10,8)*3 then ‘3억이하’

?10*10*10*10*10*10*10*10 (10의8 제곱근)

when amt<=porwer (10,9)*1 then ‘10억이하’

when amt<=porwer (10,10)*1 then ‘100억이하’

when amt<=porwer (10,10)*5 then ‘500억이하’

when amt>porwer (10,10)*5 then ‘500억초과’

END GUBUN ,

TO_CHAR (AMT,’fm999,999,999,999,999’)amt01 ?max 기준치로 금액단위 구분(,) 나누기

From (

Select

Mer_no,--가맹점번호

Sum(일시불매입금액+할부매입금액+즉시불매입금액+선불카드매입금액)amt

From month_12_mer_total

Where standard_date=’202203’

Group by mer_no

)

;

 

1-2 가맹점등급별 금액단위 별 승인/매출액 및 매출건수 산출

Create table hk_pay_cate_on_off_amt_2205 as

select

/*+full(a) parallel (a 8) */

b.gubun ,

A, reg_yymm, ?등록월

,sum(sale_amt) amt ?매출금액

,Sum(case when sale_ctgo_cd like ‘0%’ THEN 1 ELSE -1 END end) cnt ?매출코드가 정상인것만

From day_card_seles a, hk_pay_cate_2205 b

Where a.mer_no=b.mer_n

And a.reg_yymm between ‘202201’ and ‘202203’

And a.mb_no=’008’?제휴사코드

And cumuse_sale_ctgo_cd < 4 (자사매출)

Group by b.gubun ,A. reg_yymm

;

 

1-3 온 오프라인 건수 산출

Create table hk_pay_cate_on_off_amt_2205 as

 

Select

gubun,

reg_yymm,

sum(amt_on) amt_on,

sum(cnt_on) cnt_on,

sum(amt_off) amt_off,

sum(cnt_off) cnt_off

from(

select

/*+full(a) parallel (a 8) */

b.gubun ,

A, reg_yymm, ?등록월

NVL(Sum ( case when A.MER_TPBUZ_NO IN (‘9720’,’9302’) THEN A.SALE_AMT END) ,0 ) amt_on,

 MER_TPBUZ_NO 가맹점 분류코드,

NVL(Sum ( case when A.MER_TPBUZ_NO IN (‘9720’,’9302’) AND SALE_GUBUN_CD LIKE ‘0%’ THEN 1

When A.MER_TPBUZ_NO IN (‘9720’,’9302’) AND SALE_GUBUN_CD LIKE ‘1%’ THEN 1 ELSE 0 END) ,0 ) cnt_on,

NVL(Sum ( case when A.MER_TPBUZ_NO not IN (‘9720’,’9302’) THEN A.SALE_AMT END) ,0 ) amt_off,

NVL(Sum ( case when A.MER_TPBUZ_NO not IN (‘9720’,’9302’) AND SALE_GUBUN_CD LIKE ‘0%’ THEN 1

When A.MER_TPBUZ_NO not IN (‘9720’,’9302’) AND SALE_GUBUN_CD LIKE ‘1%’ THEN 1 ELSE 0 END) ,0 ) cnt_off

From day_card_seles a, hk_pay_cate_on_off_amt_2205 b

Where a.mer_no=b.mer_n

And a.reg_yymm between ‘202201’ and ‘202203’

And a.mb_no=’008’ 제휴사코드

And cumuse_sale_ctgo_cd < 4 (자사매출)

Group by b.gubun ,A. reg_yymm

Order by 1,2 asc

) a

Group by gubun , reg_yymm

;

 

7.

프로젝트명: 대구은행 거리두기 해제 전후 종류별(체크/신용/선불/지역화폐) 매출액 현황 자료 요청

작성시점: 2022.05.25~2022.05.27(2영업일)

핵심목표: 거리두기 해제 전/후 비교를 통한 언론보도자료 활용 시 현황 자료로 사용

요청대상: 대구은행

요청내용: 대구시/포항시의 거리두기 해제 전 (2021.12~2022.02) 해제 후 (2022.04~05) 기준 

체크/신용/선불/지역화폐 매출액 및 이용건수 현황 산출

- 매출구분: 자사 회선이용 정회원사, 대구은행

- 산출방식: 지역 가맹점별 신용/체크/선불/지역화폐 실적 산정

기간 산정 시 선불은 승인일자 기준으로 그외에는 매출일자 기준으로 산출

-기타사항: 법인의 경우 선불/지역화폐(포항/대구)에 대한 건수와 금액이 없음으로 서식에서 제외  



<관련쿼리>



/*가맹점 산출*/



CREATE TABLE hk_2022_sheet1 as

SELECT

/*+Full(a) PARALLEL(a 8) Full(b) PARALLEL(b 8)  Full(c) PARALLEL(c 8) COMPRESS NOLOGGING (a+b+c)*/

'대구' gb

, a.MER_NO as mer_no

, b.TP_BUZ_NO as mer_tpbuz_no

, b.TP_BUZ_NM

, b.TP_GRP_NO

, b.TP_GRP_NM

, c.SIDO_NM

, c.CCG_NM

, c. ADNG_NM

FROM ADE.TWDDMERBS A, ODW.TOZMERTBZ B

,( SELECT distinct a.rgn_nm

, b.sido_nm

, b.CCG_NM

, b.ADNG_NM

, b.ZP

FROM CRD.TBZCCGRN A, CRD.TBZZPBD B

WHERE substr(B.ZP,1,3) = a.mail_rgn_no

and a.zp_typ_cd='03'

and substr(a.rgn_no,1,5) like '대구%'

and sido nm='대구'

) c

WHERE 1=1

and a.MER_TPBUZ_NO = b.TP_BUZ_NO(+) --left join

and a.BIZL_ZP = c.ZP



UNION



SELECT

/*+Full(a) PARALLEL(a 8) Full(b) PARALLEL(b 8)  Full(c) PARALLEL(c 8) COMPRESS NOLOGGING (a+b+c)*/

'포항' gb

, a.MER_NO as mer_no

, b.TP_BUZ_NO as mer_tpbuz_no

, b.TP_BUZ_NM

, b.TP_GRP_NO

, b.TP_GRP_NM

, c.SIDO_NM

, c.CCG_NM

, c. ADNG_NM

FROM ADE.TWDDMERBS A, ODW.TOZMERTBZ B

,( SELECT distinct a.rgn_nm

, b.sido_nm

, b.CCG_NM

, b.ADNG_NM

, b.ZP

FROM CRD.TBZCCGRN A, CRD.TBZZPBD B

WHERE substr(B.ZP,1,3) = a.mail_rgn_no

and a.zp_typ_cd='03'

and substr(a.rgn_no,1,5) like '포항%'

and sido nm='경북'

) c

WHERE 1=1

and a.MER_TPBUZ_NO = b.TP_BUZ_NO(+) --left join

and a.BIZL_ZP = c.ZP 

;



/*카드구분 별 항목 분류*/--대용량데이터

CREATE TABLE hk_2022_sheet2 as

SELECT

/*+Full(a) PARALLEL(a 16) Full(b) PARALLEL(b 16)  COMPRESS NOLOGGING (a+b)*/

b.gb

, a.INDV_CP_DV_CD

, a.MB_NO

, decode (a.CR_CSALE_CTGO_CD,'0','credit','1','check','2','debt','credit')SALE_GB

, case when a.CR_CSALE_CTGO_CD = '3' then a.SALE_DATE 

else a.AUTH_DATE end NEW_date

, case when a.CARD_PDCT_NO in('123456','765432') then '대구화폐'

    when a.CARD_PDCT_NO in('928399','274930') then '포항화폐'

    else Null end SALE_GB2

, b.TP_BUZ_NO

, b.TP_GRP_NM

, b.SIDO_NM

, b.CCG_NM

, b.ADNG_NM

, sum(a.SALE_AMT) as amt

, sum(case when a.SALE_CTGO_CD like '0%' then 1 else -1 end) as cnt

FROM ADE.TWBSALED A, hk_2022_sheet1 B

WHERE a.MER_NO = b.MER_NO

and a.CMUSE_SALE_CTGO_CD<'2'-- 자사 매출

and  a,MER_TPBUZ_NO not in ('9292')

and a.MB_KND_CD='1'-- 정회원사

and a.REG_YYMM between '202112' and '202205'

GROUP BY b.gb

, a.INDV_CP_DV_CD

, a.MB_NO

, decode (a.CR_CSALE_CTGO_CD,'0','credit','1','check','2','debt','credit')

, case when a.CR_CSALE_CTGO_CD = '3' then a.SALE_DATE 

else a.AUTH_DATE end

, case when a.CARD_PDCT_NO in('123456','765432') then '대구화폐'

    when a.CARD_PDCT_NO in('928399','274930') then '포항화폐'

    else Null end

, b.TP_BUZ_NO

, b.TP_GRP_NM

, b.SIDO_NM

, b.CCG_NM

, b.ADNG_NM

;



/*체크/신용/선불/지역화폐 분류*/



CREATE TABLE hk_2022_sheet3 COMPRESS NOLOGGING as

SELECT 

gb

, INDV_CP_DV_CD

, MB_NO

, case when SALE_GB2 is null then sale_gb

when SALE_GB2 is not null then SALE_GB2 end sale_gb

, NEW_DATE

,TP_BUZ_NM

,TP_GRP_NM

,SIDO_NM

,CCG_NM

,ADNG_NM

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet2

GROUP BY gb

, INDV_CP_DV_CD

, MB_NO

, case when SALE_GB2 is null then sale_gb

when SALE_GB2 is not null then SALE_GB2 end

, NEW_DATE

,TP_BUZ_NM

,TP_GRP_NM

,SIDO_NM

,CCG_NM

,ADNG_NM

;



/*지역별 분류*/

CREATE TABLE hk_2022_sheet4 COMPRESS NOLOGGING as

SELECT 

/*+Full(a) PARALLEL(a 16) */

'1-1.대구전체' gb_1

, INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END term_gb

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

, SIDO_NM || ' ' || CCG_NM || ' ' || ADNG_NM as zp_gb

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet3

WHERE gb='대구' -- 지역지정 전체, 대구은행 구분

GROUP BY INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

,SIDO_NM

,CCG_NM

,ADNG_NM




UNION ALL





SELECT 

/*+Full(a) PARALLEL(a 16) */

'1-2.대구은행' gb_1

, INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END term_gb

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

, SIDO_NM || ' ' || CCG_NM || ' ' || ADNG_NM as zp_gb

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet3

WHERE MB_NO= '001' -- 대구은행

and gb='대구' -- 지역지정 전체, 대구은행 구분

GROUP BY INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

,SIDO_NM

,CCG_NM

,ADNG_NM





UNION ALL







SELECT 

/*+Full(a) PARALLEL(a 16) */

'2-1.포항전체' gb_1

, INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END term_gb

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

, SIDO_NM || ' ' || CCG_NM || ' ' || ADNG_NM as zp_gb

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet3

WHERE gb='포항' -- 지역지정 전체, 대구은행 구분

GROUP BY INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

,SIDO_NM

,CCG_NM

,ADNG_NM







UNION ALL



 

SELECT 

/*+Full(a) PARALLEL(a 16) */

'2-2.대구은행_포항' gb_1

, INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END term_gb

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

, SIDO_NM || ' ' || CCG_NM || ' ' || ADNG_NM as zp_gb

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet3

WHERE MB_NO= '001' -- 대구은행

and gb='포항' -- 지역지정 전체, 대구은행 구분

GROUP BY INDV_CP_DV_CD

, case when NEW_DATE between  '20211201' and '20220225'  then '전'

when NEW_DATE between  '20220401' and '20220531'  then '후' 

ELSE null END

, SALE_GB

, TP_BUZ_NM

, TP_GRP_NM

,SIDO_NM

,CCG_NM

,ADNG_NM



;





/*자사 로고 회원사 기준 대구/포항 지역 카드매출 */

SELECT 

gb1

, INDV_CP_DV_CD

,term_gb

,SALE_GB

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet4

WHERE term_gb is not null

GROUP BY gb1

, INDV_CP_DV_CD

,term_gb

,SALE_GB

ORDER BY 1,2,3,4 asc

;



/*자사 로고 회원사 기준 대구/포항 지역 가맹점별 카드매출 */

SELECT 

gb1

, TP_BUZ_NM

, TP_GRP_NM

, INDV_CP_DV_CD
, term_gb

,SALE_GB

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet4

WHERE term_gb is not null

GROUP BY gb1

, TP_BUZ_NM

, TP_GRP_NM

, INDV_CP_DV_CD
, term_gb

,SALE_GB

;



/*자사 로고 회원사 기준 대구/포항 지역 가맹점별 카드매출 동향 분석자료(지역(동)별)*/

SELECT 

gb1

, zp_gb
, INDV_CP_DV_CD
, term_gb

,SALE_GB

,sum(AMT) as AMT

,sum(CNT) as CNT

FROM hk_2022_sheet4

WHERE term_gb is not null

GROUP BY gb1
, zp_gb
, INDV_CP_DV_CD
, term_gb

,SALE_GB

;

