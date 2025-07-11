# ML_FINAL
# Walmart-ის გაყიდვების პროგნოზირება

## Kaggle-ის კონკურსის მოკლე მიმოხილვა

პროექტის მიზანია **Walmart Recruiting - Store Sales Forecasting** Kaggle-ის კონკურსის ფარგლებში Walmart-ის 45 მაღაზიის სხვადასხვა დეპარტამენტის ყოველკვირეული გაყიდვების პროგნოზირება. მოდელის შეფასების მთავარი მეტრიკაა **Weighted Mean Absolute Error (WMAE)**, სადაც სადღესასწაულო კვირებს 5-ჯერ მეტი წონა ენიჭება.

## ჩვენი მიდგომა პრობლემის გადასაჭრელად

პრობლემის გადასაჭრელად გამოვიყენეთ რამდენიმე განსხვავებული მიდგომა, დაწყებული კლასიკური სტატისტიკური მოდელებიდან, დასრულებული თანამედროვე Deep Learning არქიტექტურებით. პროცესი მოიცავდა შემდეგ ეტაპებს:

1.  **საბაზისო მოდელების შექმნა:** თავდაპირველად შევქმენით მარტივი მოდელები (Linear Regression, Decision Tree) და კლასიკური დროითი მწკრივის მოდელი (ARIMA) შედეგების საწყისი ნიშნულის დასაფიქსირებლად.
2.  **ხეზე დაფუძნებული მოდელების ტესტირება:** გავტესტეთ უფრო მძლავრი მოდელები, როგორიცაა Random Forest, LightGBM და XGBoost. ამ ეტაპზე აქტიურად გამოვიყენეთ Feature Engineering ტექნიკები.
3.  **Deep Learning მოდელების გამოყენება:** გამოვიკვლიეთ თანამედროვე ნეირონული ქსელები, რომლებიც სპეციალურად დროითი მწკრივებისთვისაა შექმნილი. მათ შორის: N-BEATS, PatchTST და AutoGluon-ის მეშვეობით გაშვებული Chronos და DeepAR.
4.  **ჰიპერპარამეტრების ოპტიმიზაცია:** საუკეთესო შედეგის მქონე მოდელებისთვის ჩავატარეთ ჰიპერპარამეტრების სისტემატური ტიუნინგი.
5.  **საბოლოო მოდელის შერჩევა:** შევადარეთ ყველა მოდელის ვალიდაციის შედეგი და შევარჩიეთ საუკეთესო, რომელიც სრულ მონაცემთა ბაზაზე დავასწავლეთ.

ყველა ექსპერიმენტი და შედეგი აღირიცხა **Weights & Biases** პლატფორმის გამოყენებით.

## რეპოზიტორიის სტრუქტურა

რეპოზიტორია შეიცავს `notebooks` დირექტორიას, რომელშიც მოთავსებულია ექსპერიმენტების Jupyter Notebook ფაილები.

-   `notebooks/arima_based.ipynb` - ARIMA/SARIMAX-ის საბაზისო მოდელი.
-   `notebooks/n_beats.ipynb`, `notebooks/2_N_Beats.ipynb` - N-BEATS მოდელის იმპლემენტაცია და ტიუნინგი.
-   `notebooks/Patch_Time_Series_Transformer.ipynb` - PatchTST მოდელი ჰიპერპარამეტრების ოპტიმიზაციით.
-   `notebooks/walmart_sales_forecasting_with_lightgbm.ipynb` - LightGBM მოდელი.
-   `notebooks/xgboost_based.ipynb`, `notebooks/walmart_sales_forecast_by_xgb_boost_ml_model.ipynb` - XGBoost-ზე დაფუძნებული მოდელი და ექსპერიმენტები.
-   `notebooks/AutoGluon_TimeSeriesPredictor.ipynb` - AutoGluon-ის გამოყენება დროითი მწკრივებისთვის.
-   `notebooks/walmart_sales_linear_dt_rf_regression.ipynb` - წრფივი, გადაწყვეტილების ხის და Random Forest მოდელები.

## Feature Engineering

მონაცემთა გამდიდრების მიზნით, გამოვიყენეთ შემდეგი მეთოდები:

#### კატეგორიული ცვლადების რიცხვითში გადაყვანა

*   **One-Hot Encoding:** `Type` (მაღაზიის ტიპი: 'A', 'B', 'C') და `HolidayType` (დღესასწაულის ტიპი) სვეტები გადავიყვანეთ რიცხვით ფორმატში ამ მეთოდით.
*   **Integer/Boolean Conversion:** `IsHoliday` სვეტი, რომელიც `True/False` მნიშვნელობებს შეიცავდა, გადავიყვანეთ 1/0 ფორმატში.

#### Nan მნიშვნელობების დამუშავება

*   **ნულით შევსება:** `MarkDown` სვეტები შეიცავდა დიდი რაოდენობით NaN მნიშვნელობებს, რაც იმას ნიშნავს, რომ იმ პერიოდში ფასდაკლება არ ყოფილა. შესაბამისად, ისინი შევავსეთ ნულით.
*   **საშუალოთი შევსება:** `CPI` და `Unemployment` სვეტებში არსებული მცირე რაოდენობის NaN-ები შევავსეთ შესაბამისი სვეტის საშუალო არითმეტიკულით.
*   **ჯგუფური საშუალოთი შევსება:** `LagAdder` ტრანსფორმერის გამოყენებისას, შექმნილი `lag` მახასიათებლების NaN-ები შევავსეთ კონკრეტული `Store-Dept` ჯგუფის საშუალო გაყიდვებით.

#### Cleaning მიდგომები

*   **Outlier-ების დამუშავება:** რიცხვითი სვეტებიდან (`Temperature`, `Fuel_Price` და სხვ.) ამოვარდნილი მნიშვნელობების დასამუშავებლად გამოვიყენეთ **Interquartile Range (IQR)** მეთოდი. ის მნიშვნელობები, რომლებიც სცდებოდა `Q1 - 1.5 * IQR` და `Q3 + 1.5 * IQR` საზღვრებს, ჩავანაცვლეთ შესაბამისი საზღვრის მნიშვნელობით.
*   **არასაჭირო სვეტების წაშლა:** მაღალი კორელაციის და VIF (Variance Inflation Factor) მაჩვენებლის გამო, `Size` სვეტი ამოვიღეთ მონაცემებიდან, რადგან მისი ინფორმაცია დუბლირებული იყო `Type` სვეტში.

## Feature Selection

#### გამოყენებული მიდგომები და მათი შეფასება

1.  **Variance Inflation Factor (VIF):** გამოვიყენეთ მულტიკოლინეარობის შესამოწმებლად. VIF-მა დაადასტურა მაღალი კორელაცია `Size` და `Type` სვეტებს შორის, რის საფუძველზეც ამოვიღეთ `Size` სვეტი.
2.  **XGBoost Feature Importance:** ხეზე დაფუძნებული მოდელების (XGBoost, RandomForest) ვარჯიშის შემდეგ, გავაანალიზეთ `feature_importances_` ატრიბუტი. ამან დაგვანახა, რომ `Dept`, `Size`, `WeekOfYear` და `CPI` ყველაზე მნიშვნელოვანი მახასიათებლები იყო.
3.  **AutoGluon Feature Importance:** AutoGluon-ის მიერ დაგენერირებულმა feature importance-მა ასევე გამოავლინა დროითი (`Days_since_last_holiday`, `WeekOfYear`) და გეოგრაფიული (`Store`, `Dept`) მახასიათებლების მაღალი მნიშვნელობა.

## Training

#### ტესტირებული მოდელები

შედარდა რამდენიმე ტიპის მოდელი:
1.  **სტატისტიკური მოდელები:** ARIMA.
2.  **ხეზე დაფუძნებული მოდელები:** Linear Regression, Decision Tree, Random Forest, LightGBM, XGBoost.
3.  **Deep Learning მოდელები:** N-BEATS, PatchTST, AutoGluon (Chronos, DeepAR).

#### Hyperparameter ოპტიმიზაციის მიდგომა

საუკეთესო შედეგების მისაღწევად გამოვიყენეთ **თანმიმდევრული Grid Search** მიდგომა. პროცესი ასე წარიმართა:
1.  დავფიქსირეთ ძირითადი პარამეტრები (`h`, `max_steps`).
2.  მონაცვლეობით მოვახდინეთ თითოეული მნიშვნელოვანი ჰიპერპარამეტრის ტიუნინგი: `input_size`, `batch_size`, `learning_rate` და ა.შ.
3.  ყოველ ეტაპზე ვარჩევდით საუკეთესო მნიშვნელობას და ვიყენებდით მას მომდევნო პარამეტრის ტიუნინგისას.

ეს მეთოდი დეტალურად არის ნაჩვენები `Patch_Time_Series_Transformer.ipynb` ფაილში.

#### საბოლოო მოდელის შერჩევის დასაბუთება

ქვემოთ მოცემულია ვალიდაციის მონაცემებზე მიღებული WMAE ქულები სხვადასხვა მოდელისთვის:

| მოდელი | ვალიდაციის WMAE |
| :--- | :--- |
| **PatchTST** | **1526.46** |
| AutoGluon (Ensemble) | 2042.61 |
| Random Forest | 2351.10 |
| N-BEATS | 2462.03 |
| XGBoost | 2848.69 |
| LightGBM | 4817.83 |
| ARIMA | 17585.58 |

შედეგებიდან ნათლად ჩანს, რომ **PatchTST** მოდელმა აჩვენა საუკეთესო შედეგი და მნიშვნელოვნად აჯობა როგორც კლასიკურ სტატისტიკურ, ისე ხეზე დაფუძნებულ მოდელებს. მისი უპირატესობაა დროით მწკრივებში რთული და გრძელვადიანი დამოკიდებულებების აღქმის უნარი, რისთვისაც ის იყენებს "პაჩებად" დაყოფილ მონაცემებს.

სწორედ ამიტომ, საბოლოო მოდელად შეირჩა **PatchTST**.

## Wandb Tracking

ექსპერიმენტების აღრიცხვის, მოდელების ვერსიონირებისა და შედეგების ვიზუალიზაციისთვის გამოვიყენეთ Weights & Biases.

*   **Wandb ექსპერიმენტების ბმული:** [Walmart Recruiting - Store Sales Forecasting](https://wandb.ai/lchik22-free-uni/Walmart%20Recruiting%20-%20Store%20Sales%20Forecasting)

#### ჩაწერილი მეტრიკების აღწერა

-   **`val_wmae` (Validation Weighted Mean Absolute Error):** მთავარი შეფასების მეტრიკა, რომელიც ითვლის შეწონილ საშუალო აბსოლუტურ ცდომილებას ვალიდაციის მონაცემებზე. სადღესასწაულო კვირებს ენიჭება წონა 5, ხოლო დანარჩენს – 1.
-   **`train_wmae`:** იგივე მეტრიკა, დათვლილი სატრენინგო მონაცემებზე.
-   **`r2_score`:** R-კვადრატის კოეფიციენტი, რომელიც აჩვენებს, თუ რამდენად კარგად ხსნის მოდელი მონაცემების ვარიაციას.
-   **Model Config:** ყოველი ექსპერიმენტისთვის შენახულია გამოყენებული ჰიპერპარამეტრები (`n_estimators`, `learning_rate`, `max_depth` და ა.შ.).
-   **Model Artifacts:** საუკეთესო მოდელები და მათი ვერსიები შენახულია არტეფაქტების სახით, რაც ამარტივებს მათ შემდგომ გამოყენებას.

#### საუკეთესო მოდელის შედეგები

-   **მოდელი:** PatchTST
-   **ვალიდაციის WMAE:** **1526.46**
-   **ჰიპერპარამეტრები:**
    -   `input_size`: 52
    -   `h`: 53
    -   `dropout`: 0.2
    -   `batch_size`: 64
    -   `activation`: 'relu'