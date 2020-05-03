# T-SQL 學習筆記 

> 筆記一下一些基本的T-SQL語法, 還有一些常用的東西, 免得又忘記...囧

<br>

Transact-SQL（又稱T-SQL）是具有批次與區塊特性的SQL指令集合，資料庫開發人員可以利用它來撰寫資料部份的商業邏輯（Data-based Business Logic），<br>
以強制限制前端應用程式對資料的控制能力。<br>
同時，它也是資料庫物件的主要開發語言。<br>

<br>

## 基本語法

+ SELECT   想要查詢的欄位
+ FROM	    想要查詢的表格
+ WHERE    查詢條件 [AND|OR] 
+ GROUP BY 分組設定
+ HAVING   分組條件
+ ORDER BY 排序設定
+ LIMIT    限制設定
<br>--------------------------------<br>
+ AVG (平均)
+ COUNT (計數, NULL時，該筆記錄不會被計算進去)
+ DISTINCT (找出不同資料)
+ IS NOT NULL
+ MAX (最大值)
+ MIN (最小值)
+ SUM (總合)
+ Len (長度)
+ Replace (改變內容)
+ DATEDIFF (算日期間的間隔，傳回帶正負號的整數)
+ ISNULL ( 有資料時傳回 , 前面為NULL 時所傳回 )

<br>

## 一、 一般查詢

<pre>

以下是可能使用到的運算元：

  1. 數字運算元
    +  -  *  /  %
  2. 比較運算元
    =  >  <  >=  <=  <>
    <strong>注意!!! 不要使用!=</strong>
  3. 邏輯運算元
    AND  OR   NOT
  4. 字串運算元
    +
</pre>

[效能優化1](http://www.cc.ntu.edu.tw/chinese/epaper/0031/20141220_3109.html)

[效能優化2](http://blog.xuite.net/j2ee/code/15120677-%E8%AA%BF%E6%A0%A1+SQL+%E4%BB%A5%E5%BE%B9%E5%BA%95%E6%94%B9%E5%96%84%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E6%95%88%E8%83%BD)

<br>

### 查詢語法範例：

<pre>

  SELECT		*
  FROM		dbo.A00B0_JobCode
  ORDER BY	Job_Code      -- 預設為升冪
  ORDER BY	Job_Code DESC -- 降冪


--當查詢結果的欄位過多時, 建議指定需要的欄位即可

  SELECT		User_Id, User_Name, Company_Code, Unit_Code 
  FROM		dbo.A00B2_UserData


-- 等於、不等於，語法範例：

  SELECT		*
  FROM		dbo.A00B1_Company
  WHERE		Company_Code = N'01'
  WHERE		Company_Code <> N'01'


-- 大於、小於，語法範例：

  SELECT		Input_Time, *
  FROM		dbo.A00B1_Company
  WHERE		Input_Time >= '2006/1/1'
  AND			Input_Time <= '2008/6/11'


  SELECT		*
  FROM		dbo.A00B1_Company
  WHERE		Company_Code IN ('01', '02') --將知道的值都放入


  SELECT		Input_Time, *
  FROM		dbo.A00B1_Company
  WHERE		Input_Time BETWEEN '2006/1/1' AND '2008/6/11'


-- LIKE、NOT LIKE，語法範例：

  SELECT		*
  FROM		dbo.A00B1_Company
  WHERE		Company_Code LIKE N'0%'
  WHERE		Company_Code NOT LIKE N'0%'

</pre>

<br>

### JOINS，語法範例：

<pre>

-- JOIN  查詢結果只會返回符合連接條件的資料

  SELECT		*
  FROM		dbo.A00B1_Company	A
  JOIN		dbo.A00B1_UnitCode	B ON A.Company_Code = B.Company_Code


-- LEFT JOIN 會返回[資料表1]中所有資料列，就算沒有符合連接條件，而[資料表2]中如果沒有匹配的資料值就會顯示為「NULL」

    SELECT		A.Company_Code, A.Company_Name, B.*
    FROM		dbo.A00B1_Company	A
    LEFT JOIN	dbo.A00B1_UnitCode	B ON A.Company_Code = B.Company_Code
    --WHERE		B.Company_Code IS NULL
    
-- Right Join 串聯兩個資料表中對應欄資料時，以[資料表2]的資料為主，若資料存在於[資料表2]，但[資料表1]沒有對應值時，仍顯示[資料表2]中的資料。
 
</pre>

<img class="header-picture" src="/images/Join.png" alt=""/>

<br>
<br>

### COUNT、GROUP BY、別名，語法範例：


幫一般欄位取一個欄位別名是比較沒有必要的，如果是運算式的話，通常就要幫它取一個欄位別名來取代原來一大串的運算式。<br>
取欄位別名的時候要特別注意２種狀況, 一定要使用單引號或雙引號, 否則執行描述以後會發生錯誤...<br>
1. 如果欄位別名包含空白, 例如 Company Nos
2. 如果堅持用SQL語法中的保留字來當作欄位別名, 例如 SELECT


<pre>

--若欄位值為NULL，則該筆記錄不會被COUNT計算進去
  SELECT		Company_Code, COUNT(*) AS Company_Count, SUM(1) AS 'Company_Nos'
  FROM		dbo.A00B1_OrgCode
  GROUP BY	Company_Code

</pre>

<br>

## 二、新增、修改與刪除

<pre>

-- 新增，兩種常用語法，語法範例：

  INSERT INTO	dbo.A00B0_JobCode
        (Job_Code, Job_Name, Biz_Code, Input_Name, Input_Time)
  VALUES	(N'IT-X1', N'測試', 'IT', N'建檔者', GETDATE())


  INSERT INTO	A00B0_JobCode  --利用子查詢，從其它的資料表中取得資料來作一次多筆新增
        (Job_Code, Job_Name, Biz_Code, Input_Name, Input_Time)
  SELECT	Job_Code, Job_Name, SUBSTRING(Job_Code,1,2), N'建檔者', GETDATE()
  FROM		dbo.A00B2_UserData
  WHERE		User_Id = N'001234'


-- 修改，兩種常用語法，語法範例：

  UPDATE dbo.A00B1_UnitCode
    SET	Company_Code = N'01',
        Company_Name = N'Github'
  WHERE	Unit_Code = N'0199'


  UPDATE	dbo.A00B1_UnitCode
  SET			Update_Name		= N'修改者',
          Update_Time		= GETDATE()
  FROM		dbo.A00B1_UnitCode A, dbo.A00B1_Company B
  WHERE		A.Company_Code = B.Company_Code
          AND	A.Unit_Code = N'001234'


-- 刪除，語法範例：

  DELETE		dbo.A00B1_UnitCode
  WHERE		Unit_Code = N''
</pre>

<br>

## 三、變數


使用者自訂變數以DECLARE宣告<br>
變數名稱以@開頭<br>
以SET或SELECT指定變數值<br>
請勿使用兩個@@，此為系統變數。列如@@error<br>


<pre>

-- 語法範例：

  DECLARE		@ExampleStr		nvarchar(4),
          @ExampleNumber	int,
          @ExampleDate	datetime,
          @ExampleBoolean	bit
  SET			@ExampleStr		= N'0113'
  SET			@ExampleNumber	= 100
  SET			@ExampleDate	= getdate()
  SET			@ExampleBoolean	= 1

  SELECT		@ExampleStr, @ExampleNumber, @ExampleDate, @ExampleBoolean

  SELECT		Company_Code, Unit_Code, Head_OnJob, Input_Time, Cancel_Mark, *
  FROM		  dbo.A00B1_OrgCode
  WHERE		  Org_Code = '01-01-0113'

  SELECT		@ExampleStr		= Company_Code + N'-' + Unit_Code,
            @ExampleNumber	= Head_OnJob + 1,
            @ExampleDate	= Input_Time,
            @ExampleBoolean	= Cancel_Mark
  FROM		  dbo.A00B1_OrgCode
  WHERE		  Org_Code = '01-01-0113'

  SELECT		@ExampleStr, @ExampleNumber, @ExampleDate, @ExampleBoolean

</pre>

<br>

## 四、判斷式

<pre>

-- IF ELSE，語法範例：

  DECLARE		@Company_Code	nvarchar(2)
  SET			@Company_Code	= N'*'

  IF ( @Company_Code = N'*' )
    BEGIN
      SELECT	*
      FROM	dbo.A00B1_Company
    END
  ELSE
    BEGIN
      SELECT	*
      FROM	dbo.A00B1_Company
      WHERE	Company_Code = @Company_Code
    END


--另一種寫法
  SELECT		*
  FROM		dbo.A00B1_Company
  WHERE		@Company_Code = N'*' OR Company_Code = @Company_Code


-- CASE END，語法範例：
  SELECT	CASE
              WHEN Company_Code LIKE '0%' THEN '台北'
              WHEN Company_Code LIKE '1%' THEN '北區'
              WHEN Company_Code LIKE '2%' THEN '中區'
              WHEN Company_Code LIKE '3%' THEN '南區'
              ELSE N'其他'
          END AS 'Region_Name',
          *
  FROM		dbo.A00B1_Company
  WHERE		Cancel_Mark = 0

</pre>

<br>

## 五、交易(Transaction)

1. BEGIN：開啟交易，打開交易功能。
2. COMMITN：確認交易，在交易結束時確認交易，在確認時資料才會真的寫入資料表。
3. ROLLBACK：回復交易，執行這行時，會回復在交易內所有T-SQL所更動的內容。

```SQL
--先開啟交易
BEGIN TRANSACTION

-- 新增、修改、刪除相關語法執行
xxx

--沒問題就確認交易完成，資料也會在這時候確認新增和修改
COMMIT TRANSACTION

--若是有問題就取消交易，讓資料不被這三段T-SQL所影響
ROLLBACK TRANSACTION

```


## 六、其他運用


### 字串

<pre>

-- 取得字串的某一段，語法範例：
    SELECT		SUBSTRING(N'01234567890',1,5)

-- 取得某個字串的位置，語法範例：
    SELECT		CHARINDEX('3',N'1234567890')


-- Split 用法

  SELECT value
  FROM dbo.fn_Split(D.Fare_Match_Route, N';')
  WHERE @Route_Summary = value
</pre>


### 時間

<pre>

-- 取出日期的特定格式，語法範例：
    SELECT		GETDATE()
    SELECT		CONVERT(nvarchar(10),GETDATE(),111) -- ex: yyyy/MM/dd
    SELECT		DATEPART(YEAR,GETDATE())


-- 日期變化，設定一個日期，取得依需求增減的日期，語法範例：
-- DATEADD 日期加上一個數值，傳回的日期

    DECLARE		@Example_Year	nvarchar(4),
              @Example_Date	datetime
    SET			  @Example_Year	= 2015
    SET			  @Example_Date	= @Example_Year + '/05/15'
    SET			  @Example_Date	= DATEADD(YEAR,1,@Example_Date)
    SELECT		@Example_Date


-- DATEDIFF 兩個日期間的間隔，傳回帶正負號的整數
    SELECT	DATEDIFF(DAY, '2010-10-03','2010-10-04'  )


-- 傳回代表指定 date 之指定 datepart 的整數
    SELECT DATEPART(year, '12:10:30.123')
    
    (DATEPART(hh ,Booking_Time) BETWEEN 12 AND 18)
</pre>

### CTE (一般資料表運算式)

<pre>

    WITH OrdersTable (單位代碼, 單位職務, 員工ID) as
    (
    Select A.Job_Code, A.Job_Name, A.Input_Name
    from dbo.A00B0_JobCode AS A
    ) SELECT * FROM OrdersTable

</pre>


### ROLLUP

<pre>

    GROUP BY ROLLUP(A.Fulfill_Date)
    ROLLUP

</pre>

### View 檢視表、視圖 (SQL View Table)

資料表是一種實體結構 (physical structure)，而 View 是一種虛擬結構 (virtual structure)。

+ 加強資料庫的安全性，View 可以將實體資料表結構隱藏起來，同時限制使用者只可以檢視及使用哪些資料表欄位。
+ 檢視表是唯讀的，亦即外部使用者無法直接透過 View 去修改內部資料。
+ 將複雜的 SQL 查詢包裝在 View 中，可以簡化查詢的複雜度。
+ 當資料表結構有變更時，只需要更改 View 的設定，不需更改程式。

### WITH (NOLOCK)

NOLOCK：指定允許中途讀取。

不會發出任何共用鎖定來防止其他交易修改目前交易所讀取的資料，其他交易所設定的獨佔鎖定也不會封鎖目前交易，使它無法讀取鎖定的資料。<br>

允許中途讀取可以提高並行程度，但代價是所讀取的資料修改後來會被其他交易<strong>回復</strong>。<br>

也就是說，若是使用WITH (NOLOCK)，是允許「中途讀取」。<br>

### ROW_NUMBER()

```sql
ROW_NUMBER() OVER(PARTITION BY AAA ORDER BY BBB)
```
OVER子句內的PARTITION BY可以指定AAA欄位做分割，被分割的會自成一個群組，並以BBB欄位下去排序編號。

## 七、其他

### 資料型態

+ char 存資料有固定長度，並且都為英文數字。
+ nchar 存資料有固定長度，但不確定是否都是英文數字。
+ varchar 存資料沒有固定長度，並且都為英文數字。
+ nvarchar 存資料沒有固定長度，且不確定是否皆為英文數字。
<br>

#### 定長(沒有var)
    文字的長度固定，當輸入的數據長度沒有達到指定的長度時將自動以英文空格在其後面填充，讓長度達到相對應的長度。

#### 變長(有var):
    表示是實際存儲空間是變長的，也就是說當輸入的數據長度沒有達到指定的長度時不會在後面填充空格。(不過text所存儲的也是可變長的)。

#### Unicode(有n):
    所有的文字都用2 Byte來儲存，即使是英文字也是使用2 Byte來儲存，就可以解決中英文字符集不兼容的問題。
    
#### 非Unicode(沒有n):
    文字是英文字符就是1 Byte;非英文字符就是使用2 Byte來儲存。


### T-SQL - 取每一群組的第一筆


[Demo](https://lwsu.com.tw/Blog/2017/06/15/20170615_SQL/)

<br>

### SQL & C# 變數對應表


<img class="header-picture" src="/images/table.png" alt=""/>

<br>

### Stored Procedure (預存程序) 優點

<img class="header-picture" src="/images/SP.png" alt=""/>

<br>


## 正規化

其目的是為了降低資料的「重覆性」與避免「更新異常」的情況發生。

### 1NF

+ 定義主鍵值（primary key）及單一值
+ 避免欄位長度無法確定、出現完全一樣的兩筆資料

### 2NF

+ 符合1NF
+ 降低資料重複性(消除相依性)

### 3NF

+ 符合1NF及2NF
+ 消除遞移相依

## 反正規化

如果過度正規化，在查詢來自於多個資料表的大量資料時，會造成效能下降。因此，若要以查詢(select)效能為考量，必須進行適當的反正規化，亦即將原來的第三正規化降成第二正規化，或是將第二正規化降成第一正規化。

+ 在做系統分析時，必須依據業務需求去做適當的資料庫規畫，若需求上是以查詢(select)效能為優先考量，資料表就不需要做太多正規化。
