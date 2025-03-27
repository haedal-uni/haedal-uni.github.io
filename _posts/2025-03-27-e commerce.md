Kaggle의 [e-commerce-customer-for-behavior-analysis](https://www.kaggle.com/datasets/shriyashjagtap/e-commerce-customer-for-behavior-analysis/data) 데이터를 가지고 진행했다.

<br><br><br><br>

## 데이터 가져오기 및 확인 
```py
import pandas as pd
df = pd.read_csv('ecommerce_customer_data_large.csv')
df.head()
# df.tail()
```
|index|Customer ID|Purchase Date|Product Category|Product Price|Quantity|Total Purchase Amount|Payment Method|Customer Age|Returns|Customer Name|Age|Gender|Churn|           
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|         
|0|44605|2023-05-03 21:30:02|Home|177|1|2427|PayPal|31|1\.0|John Rivera|31|Female|0|       
|1|44605|2021-05-16 13:57:44|Electronics|174|3|2448|PayPal|31|1\.0|John Rivera|31|Female|0|  
|2|44605|2020-07-13 06:16:57|Books|413|1|2345|Credit Card|31|1\.0|John Rivera|31|Female|0|   
|3|44605|2023-01-17 13:14:36|Electronics|396|3|937|Cash|31|0\.0|John Rivera|31|Female|0|     
|4|44605|2021-05-01 11:29:27|Books|259|4|2598|PayPal|31|1\.0|John Rivera|31|Female|0|        

평균이 중앙값보다 크거나 작은 차이가 크지 않은걸 보아 데이터가 한쪽으로 치우치는 경향이 없어보인다.

<br><br><br>

### 데이터 확인하기
```py
df.shape # (250000, 13)  행, 열 개수 확인
df.columns # 전체 column 확인
```
``` 
Index(['Customer ID', 'Purchase Date', 'Product Category', 'Product Price',
       'Quantity', 'Total Purchase Amount', 'Payment Method', 'Customer Age',
       'Returns', 'Customer Name', 'Age', 'Gender', 'Churn'],
      dtype='object')
```
kaggle에서 컬럼에 관한 정보를 확인했을 때는 12개로 나왔으나 실제 데이터는 13개가 존재했다.  

확인을 해보니 Customer Age와 Age가 존재했다.

<br><br><br>

```py
cts_age = df['Customer Age']
age = df['Age']
if cts_age.equals(age) :
  print("same")
else :
  print("diff")
```
출력 결과 동일하게 나와서 설명에 없는 Age 열을 삭제했다.

<br>

```py
df.drop('Age', axis = 1, inplace=True) # 열 삭제
```

<br><br><br>  

### 결측치 확인  
```
df.isna().sum()
```

Returns 열에만 Nan값이 있어 개수를 확인했다. 총 47382로 전체 데이터의 약 19%를 차지한다.

![image](https://github.com/user-attachments/assets/78dd2be5-9b32-4156-81a5-86e7806f36ef){: width="50%"}  

<br><br>

```py
df.dropna(inplace=True) # 결측치 제거 
df.describe()
df.info()
```
|index|Customer ID|Product Price|Quantity|Total Purchase Amount|Customer Age|Returns|Churn|     
|---|---|---|---|---|---|---|---|       
|count|202618\.0|202618\.0|202618\.0|202618\.0|202618\.0|202618\.0|202618\.0|       
|mean|25020\.140234332586|254\.8990662231391|3\.0084197850141647|2725\.817661806947|43\.817923382917606|0\.500824211077002|0\.20108776120581587|
|std|14412\.38867355113|141\.72042520520375|1\.4143385567083016|1442\.2684914912043|15\.35606675487065|0\.5000005545271854|0\.4008145037060417|
|min|1\.0|10\.0|1\.0|100\.0|18\.0|0\.0|0\.0|
|25%|12599\.25|133\.0|2\.0|1478\.0|30\.0|0\.0|0\.0|
|50%|25018\.0|255\.0|3\.0|2727\.0|44\.0|1\.0|0\.0|
|75%|37444\.0|377\.0|4\.0|3975\.0|57\.0|1\.0|0\.0|
|max|50000\.0|500\.0|5\.0|5350\.0|70\.0|1\.0|1\.0|


```
<class 'pandas.core.frame.DataFrame'>
Index: 202618 entries, 0 to 249999
Data columns (total 12 columns):
 #   Column                 Non-Null Count   Dtype  
---  ------                 --------------   -----  
 0   Customer ID            202618 non-null  int64  
 1   Purchase Date          202618 non-null  object 
 2   Product Category       202618 non-null  object 
 3   Product Price          202618 non-null  int64  
 4   Quantity               202618 non-null  int64  
 5   Total Purchase Amount  202618 non-null  int64  
 6   Payment Method         202618 non-null  object 
 7   Customer Age           202618 non-null  int64  
 8   Returns                202618 non-null  float64
 9   Customer Name          202618 non-null  object 
 10  Gender                 202618 non-null  object 
 11  Churn                  202618 non-null  int64    
 ```

<br><br><br><br>

## 시각화
### 연령대 변환 및 날짜 처리 
기본적인 데이터를 가지고 시각화를 진행했다. 

성별, 연령대, 반품, 이탈, 제품, 날짜를 시각화 했으며 연령을 연령대로 변환해 진행했다.

- `plt.xticks()` : x축에 간격을 구분하기 위해 표시하는 눈금    
- `plt.xlim()` : x축이 표시되는 범위를 지정하거나 반환     
```py
# 제품 카테고리 확인  
df['Product Category'].unique() # 'Home', 'Electronics', 'Books', 'Clothing'

# 연령대 설정
def age_group(age):
    if age < 20:
        return "10대"
    elif age < 30:
        return "20대"
    elif age < 40:
        return "30대"
    elif age < 50:
        return "40대"
    elif age < 60:
        return "50대"
    elif age < 70:
        return "60대"
    else:
        return "70대"
df["Age_Group"] = df["Customer Age"].apply(age_group)

# 날짜 설정
df['Purchase Date'] = pd.to_datetime(df['Purchase Date'], format='%Y-%m-%d %H:%M:%S')
df['Year'] = pd.to_datetime(df['Purchase Date']).dt.year
df['Month'] = pd.to_datetime(df['Purchase Date']).dt.month
df['Hour'] = pd.to_datetime(df['Purchase Date']).dt.hour
df["Weekday"] = df["Purchase Date"].dt.day_name()
```
<br><br>

### subplot 시각화 
```py
import matplotlib
import matplotlib.pyplot as plt
matplotlib.rcParams['font.family'] = 'NanumGothic'
matplotlib.rcParams['axes.unicode_minus'] = False

fig, ax = plt.subplots(4,2, figsize=(13,19))

# 성별
gender = df['Gender'].value_counts()
ax[0][0].pie(gender, labels=gender.index, autopct='%.1f%%', startangle=90)
ax[0][0].set_title('성별')


# 연령대
age_order = ["10대", "20대", "30대", "40대", "50대","60대", "70대"]
df["Age_Group"] = pd.Categorical(df["Age_Group"], categories=age_order, ordered=True)

age_counts = df["Age_Group"].value_counts().sort_index() # value_counts() : 고유한 값의 등장 횟수   
ax[0][1].bar(age_counts.index, age_counts.values, color='skyblue', edgecolor='black')
ax[0][1].set_title('연령대')
ax[0][1].set_ylabel('Count')
for idx, val in age_counts.items() : # (index, value) 형태
  ax[0][1].text(idx, (val*1.01), str(val), ha='center') # x좌표, y좌표, 문자열 


# 반품
returns = df['Returns'].value_counts()
ax[1][0].pie(returns, labels=returns.index, autopct='%.1f%%', startangle=90)
ax[1][0].set_title("반품")


# 이탈
churn = df['Churn'].value_counts()
ax[1][1].pie(churn, labels=churn.index, autopct='%.1f%%', startangle=90)
ax[1][1].set_title('이탈')


# 제품
category = df['Product Category'].value_counts()
ax[2][0].bar(category.index, category.values, color='skyblue', edgecolor='black')
ax[2][0].set_title('제품')


# 날짜
year_category = df['Year'].value_counts().sort_index()
ax[2][1].bar(year_category.index, year_category.values, color='skyblue', edgecolor='black')
ax[2][1].set_xticks(year_category.index)  # x축 눈금을 연도별로 설정
ax[2][1].set_xticklabels(year_category.index.astype(str))  # 소수점 없이 정수 연도로 표시
ax[2][1].set_title('Year')

month_category = df['Month'].value_counts()
ax[3][0].bar(month_category.index, month_category.values, color='skyblue', edgecolor='black')
ax[3][0].set_title('Month')

hour_category = df['Hour'].value_counts()
ax[3][1].bar(hour_category.index, hour_category.values, color='skyblue', edgecolor='black')
ax[3][1].set_title('Hour')
```

![image](https://github.com/user-attachments/assets/06c7e0dc-31ed-4720-a1ad-7484ef2e967a)

- **성별** : 남녀의 비율은 거의 비슷하게 분포 
- **연령대** : 20대~60대는 고르게 분포되었으며 10대와 70대는 상대적으로 적은 비율을 차지  
- **반품율** : 전체 구매 중 반품 비율은 50.1%로 절반 수준
- **이탈 고객** : 전체 고객 중 20.1%가 이탈한 것으로 나타남
- **제품별 구매 비율** : 전체적으로 제품이 고르게 판매됨 
- **연도별 구매**: 2020~2022년까지는 구매가 많았으나 2023년에는 구매가 감소했다. 
- **월별 구매**: 1 ~ 8월에 구매율이 높고 9 ~ 12월에는 상대적으로 낮음 
- **시간대별 구매 분포**: 전체적으로 고른 분포를 보이고 있음 

<br><br><br><br>

### seaborn
*subplot을 명시적으로 만든 경우 Axes 객체를 전달해야한다. 

#### 연령대별 평균 구매 금액 
```py
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(8, 5))
total_purchase = df.groupby("Age_Group")["Total Purchase Amount"].mean().round(1)
sns.barplot(x=total_purchase.index, y=total_purchase.values)
for idx, val in total_purchase.items() : # (index, value) 형태
  ax.text(idx, val + 30, str(val), ha='center')
plt.title("연령대별 평균 구매 금액")
plt.show()
```
![image](https://github.com/user-attachments/assets/5ed88327-519f-45a5-a4e7-039e773aea80)

연령대가 높을수록 평균 구매 금액이 증가하는 형태를 보인다.  

<br><br><br>

#### 연령대, 총 구매 금액 
```py
import plotly.express as px
co_matrix =  df[['Total Purchase Amount', 'Customer Age']].corr().round(2)
px.imshow(co_matrix, text_auto=True)
```
![image](https://github.com/user-attachments/assets/8a028d8b-bf4a-4442-88b1-1d16a0d03cf7)

연령대와 총 구매 가격의 상관계수는 0.06으로 서로 관련이 없다는 것을 보여준다. 

<br><br><br>

#### 요일 별 구매 빈도 
```py
plt.figure(figsize=(8, 5))
sns.countplot(x="Weekday", data=df, order=["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"])
plt.title("요일별 구매 건수")
plt.show()
```
![image](https://github.com/user-attachments/assets/31ea695e-213c-48c9-8ec2-f3d1c8ec5c3d)

요일별 차이가 거의 없다. 

<br><br>  

log로 변환해서 요일 별 차이를 크게 나타냈다.
```py
plt.figure(figsize=(8, 5))
sns.countplot(x="Weekday", data=df, order=["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"])
plt.title("요일별 구매 건수")
plt.yscale('log')
plt.show()
```
![image](https://github.com/user-attachments/assets/2fab447c-0f4a-4b1b-bbe2-96a65b54917d)

수, 목, 토요일 순으로 구매 건수가 높고 일요일이 가장 낮다.  

<br><br><br> 

#### 월별 카테고리 판매량
```py
month_product = df.groupby(["Month", "Product Category"])["Quantity"].sum().unstack()

# 히트맵 시각화
plt.figure(figsize=(10, 6))
sns.heatmap(month_product, cmap="Blues", annot=True, fmt=".0f") # 데이터 값 표시 : annot=True 
plt.title("월별 제품 카테고리 판매량")
plt.xlabel("제품 카테고리")
plt.ylabel("월")
plt.show()
```
![image](https://github.com/user-attachments/assets/1b2ef50d-06b0-4dac-94fb-d226883c66b0)

이전에 살펴본 월별 구매 그래프와 같이 1 ~ 8월에는 전체적으로 판매량이 높지만 

9 ~ 12월에는 판매량이 제품과 상관없이 저조해지는 것을 볼 수 있다.    

