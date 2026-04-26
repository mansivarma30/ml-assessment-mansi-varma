**B1. Problem Formulation**
**(a) ML Problem Formulation**
1. Target variable: items_sold (sales volume per store per month).

2. Candidate input features:
Store attributes: size, location type, footfall, competition density, demographics.
Promotion type (categorical: Flat Discount, BOGO, Free Gift, Category Offer, Loyalty Points).
Temporal features: month, season, festival/holiday flags, weekend indicators.

3. Problem type: Supervised regression problem.
Justification: The target is continuous (sales volume). We want to predict numeric outcomes, not classify categories.

(**b) Target Variable Choice**
1. Why items sold is better than revenue:
Revenue is influenced by pricing, discounts, and product mix, which vary across stores and promotions.
Items sold directly reflects customer response to promotions, independent of price fluctuations.

2. Broader principle: Target variables must align with the business objective and be stable, comparable, and directly measurable. Choosing the wrong target introduces noise and bias.

**(c) Modelling Strategy**
1. A single global model ignores heterogeneity across stores.

2. Alternative: Hierarchical or segmented modelling:
- Train separate models for urban, semi‑urban, and rural stores.
- Or use a multi‑task learning approach where store ID/location is an explicit feature, allowing the model to learn store‑specific effects.

3. Justification: Different demographics and competition levels mean promotions have varying effectiveness by location.

**B2. Data and EDA Strategy**
**(a) Data Integration**
1. Tables:
Transactions (date, store_id, items_sold).
Store attributes (store_id, size, location, competition density).
Promotion details (promotion_type, start/end dates).
Calendar (date, weekend flag, festival flag).

2. Join strategy: Merge on store_id and transaction_date.
3. Grain of final dataset: One row = store × month × promotion.
4. Aggregations:
Sum items_sold per store per month.
Average competition density, footfall, and promotion exposure.
Encode categorical attributes (store size, location type, promotion type).

**(b) EDA Plan**
1. Promotion effectiveness plots: Compare average items sold across promotion types. → Guides feature importance and baseline expectations.

2. Location analysis: Boxplots of items sold by urban/semi‑urban/rural. → Reveals geographic heterogeneity.

3. Seasonality trends: Line charts of items sold over months. → Identifies seasonal peaks (festivals, holidays).

4. Correlation heatmap: Between numerical features (competition density, footfall) and items sold. → Guides scaling and feature selection.
Findings from EDA inform feature engineering (e.g., interaction terms between location and promotion).

**(c) Imbalance in Promotions**
1. Issue: 80% of transactions have no promotion → model may learn “no promotion” as default and underfit promotion effects.

2. Mitigation:
Use stratified sampling or reweighting to balance promotion vs no‑promotion cases.
Oversample promotion transactions or apply techniques like SMOTE for categorical imbalance.
Evaluate models specifically on promotion subsets to ensure effectiveness is captured.

**B3. Model Evaluation and Deployment**
**(a) Train-Test Split & Metrics**
1. Split strategy: Temporal split — train on first 2.5 years, test on last 0.5 year.

2. Why not random: Random mixing would leak future information into training, inflating performance.

3. Metrics:
RMSE (Root Mean Squared Error): Penalises large errors, useful for forecasting accuracy.
MAE (Mean Absolute Error): Easier to interpret in business terms (“average error in items sold”).
R²: Explains variance captured by the model.

4. Interpretation: Lower RMSE/MAE means more reliable promotion recommendations; R² shows explanatory power.

**(b) Explaining Recommendations**
Use feature importance from models (e.g., Random Forest).

Example: Loyalty Points Bonus in December may be driven by high festival flag + strong customer loyalty segment. Flat Discount in March may be driven by competition density + lower footfall.

Communicate to marketing team with clear visuals (bar charts of top features) and narrative: “The model recommends Loyalty Points in December because customer loyalty and festival timing strongly drive sales.”

**(c) Deployment Process**
1. Model saving: Export pipeline with preprocessing + trained model using joblib.
2. Data preparation: Each month, ingest new store attributes, promotions, and calendar data → preprocess with same pipeline.
3. Prediction: Generate promotion recommendations for all 50 stores at start of month.
4. Monitoring: Track prediction errors (RMSE, MAE) and promotion effectiveness over time.
5. Retraining triggers: If performance degrades (e.g., drift in feature distributions, declining accuracy), schedule retraining with latest data.
6. Automation: Integrate into workflow so recommendations are produced consistently without manual intervention.
