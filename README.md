# 🔍 MQL 데이터 기반 B2B 영업기회 창출 예측 모델 개발  
**LG Aimers 4기 해커톤 프로젝트**

## 1. 프로젝트 개요

B2B Marketing은 **기업 고객을 대상으로 영업 기회를 발굴하고 지속적인 매출을 창출하는 것**을 목표로 한다.  

MQL(Marketing Qualified Lead) 고객을 기반으로 영업사원을 할당하고, **최종 구매로 이어지게 하는 개인화 마케팅 활동**이 진행되는데, **할당 가능한 영업 인력은 한정적**이다.

따라서 본 프로젝트에서는 MQL 고객 정보를 활용해 **영업 전환 성공 여부를 예측하는 AI 모델**을 개발하고, 이를 통해 **영업사원을 효율적으로 배치**하는 것을 목표로 한다.

> 🏅 **해커톤 결과**: 약 900명 중 상위 6%의 성과 달성  

---

## 2. 데이터셋 설명  

| bant_submit | customer_country        | business_unit | com_reg_ver_win_rate | customer_idx | customer_type        | enterprise | historical_existing_cnt | id_strategic_ver | it_strategic_ver | ...  | response_corporate | expected_timeline     | ver_cus | ver_pro | ver_win_rate_x | ver_win_ratio_per_bu | business_area       | business_subarea | lead_owner | is_converted |
|-------------|-------------------------|---------------|---------------------|--------------|----------------------|------------|------------------------|------------------|------------------|------|--------------------|-----------------------|---------|---------|----------------|-----------------------|---------------------|------------------|------------|--------------|
| 1.0         | /Quezon City/Philippines | AS            | 0.066667            | 32160        | End-Customer          | Enterprise | NaN                    | NaN              | NaN              | ...  | LGEPH              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | Engineering      | 0          | True         |
| 1.0         | /PH-00/Philippines       | AS            | 0.066667            | 23122        | End-Customer          | Enterprise | 12.0                   | NaN              | NaN              | ...  | LGEPH              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | Advertising      | 1          | True         |
| 1.0         | /Kolkata /India          | AS            | 0.088889            | 1755         | End-Customer          | Enterprise | 144.0                  | NaN              | NaN              | ...  | LGEIL              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | Construction     | 2          | True         |
| 1.0         | /Bhubaneswar/India       | AS            | 0.088889            | 4919         | End-Customer          | Enterprise | NaN                    | NaN              | NaN              | ...  | LGEIL              | less than 3 months     | 1       | 0       | 0.003079       | 0.026846              | corporate / office   | IT/Software      | 3          | True         |
| 1.0         | /Hyderabad/India         | AS            | 0.088889            | 17126        | Specifier/ Influencer | Enterprise | NaN                    | NaN              | NaN              | ...  | LGEIL              | less than 3 months     | 0       | 0       | 0.003079       | 0.026846              | corporate / office   | NaN              | 4          | True         |

- 전체 약 **60,000건**의 MQL 데이터로 구성  
- 주요 컬럼:
  - **고객 정보**
    - `customer_country`: 고객 위치
    - `customer_type`: 고객 유형 (End-Customer, Influencer 등)
    - `enterprise`: 기업 고객 여부
    - `business_area`, `business_subarea`: 산업군 및 세부 산업군

  - **제품 및 전략 정보**
    - `business_unit`: 관심 제품군
    - `com_reg_ver_win_rate`, `ver_win_rate_x`, `ver_win_ratio_per_bu`: 과거 유사 사례의 수주 성공률 관련 변수
    - `id_strategic_ver`, `it_strategic_ver`: 전략적 제품 관련 여부

  - **문의 정보**
    - `lead_owner`: 리드 담당자
    - `expected_timeline`: 예상 구매 시기
    - `response_corporate`: 담당 법인
    - `lead_description`: 고객 문의 텍스트  
  
  - **기타**
    - `customer_idx`: 고객 고유 식별자
    - `historical_existing_cnt`: 과거 거래 수 등 히스토리
  
  - **타겟 변수**
    - `is_converted`  
      - `True`: 영업 전환 성공  
      - `False`: 영업 전환 실패  

## 3. EDA (탐색적 데이터 분석)
### 1. Lead Description 텍스트 길이와 전환율 관계 분석  
고객이 작성한 `lead_description` 텍스트의 총 길이를 나타내는 `lead_desc_length` 변수를 생성하였다.  
`lead_desc_length`가 길수록 영업 전환(`is_converted`) 성공률이 높을 것이라는 가설을 세우고 시각화한 결과, 실제로 전환율이 길이가 긴 리드에서 더 높게 나타나는 경향을 확인하였다.



### 2. 고객 문의 유형별 전환율 분석  
고객의 문의 유형에 따른 전환율을 분석한 결과, `'quotation//purchase'` 카테고리에서 전환율이 특히 높게 나타났다.  
이를 통해 기존 범주형 변수의 각 카테고리별 전환율을 파생변수로 추가하면 예측 모델 성능 향상에 도움이 될 것이라는 인사이트를 얻었다.



### 3. ‘lead_desc_length’ 변수 분포 및 변환  
`lead_desc_length`의 분포가 오른쪽으로 치우쳐 있는(skewed) 것을 확인하였고, 이를 보정하기 위해 Box-Cox 변환을 수행하였다.  
변환 후에는 분포가 보다 정규분포에 가까워졌으며, 모델 학습에 더 적합한 형태가 되었다.



### 4. 파생변수와 타겟 변수 간 상관관계 분석  
각 범주형 변수의 카테고리별 전환율을 계산하여 파생변수로 생성한 뒤, 이 변수들과 타겟 변수 `is_converted` 간의 상관관계를 히트맵으로 시각화하였다.  

특히 `customer_idx_converted_rate` 변수가 `is_converted`와 0.97의 매우 강한 양의 상관관계를 보였으며, 이는 고객별 과거 전환률이 현 전환 예측에 매우 중요한 변수임을 시사하였다.

