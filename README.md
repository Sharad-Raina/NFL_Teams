### NFL Attendance and Playoff Prediction Analysis

This project investigates the relationship between weekly attendance and various factors such as team rating, margin of victory, strength of schedule, and playoff qualification. The goal was to build predictive models for weekly attendance and evaluate their performance using both linear regression and random forest models.

The key steps and methodologies are as follows:

#### 1. Data Loading and Preprocessing

The first step involved loading two datasets: **attendance.csv** and **standings.csv**, followed by a merge to join them based on key attributes like team, team name, and year. The datasets were cleaned by filtering out missing attendance values.

```r
# Loading the attendance and standings data
attendance = read_csv("/path/to/attendance.csv", show_col_types = FALSE)
standings = read_csv("/path/to/standings.csv", show_col_types = FALSE)

# Merging the datasets
attendance_joined = attendance %>%
  left_join(standings, by = c("team", "team_name", "year"))
```
<img width="913" alt="Screenshot 2025-02-07 at 6 48 34 PM" src="https://github.com/user-attachments/assets/b1076a85-fefd-4d81-9d44-3cb000cbe084" />



#### 2. Data Visualization

The merged data was analyzed with basic visualizations to identify trends and relationships:

- **Boxplot of Weekly Attendance by Playoff Status**: This showed a higher average weekly attendance for teams that qualified for the playoffs.

```r
attendance_joined %>%
  filter(!is.na(weekly_attendance)) %>%
  ggplot(aes(weekly_attendance, fct_reorder(team_name, weekly_attendance), fill = playoffs)) +
  geom_boxplot(outlier.alpha = 0.5)
```
<img width="552" alt="Screenshot 2025-02-07 at 6 47 18 PM" src="https://github.com/user-attachments/assets/08b17a34-255c-4422-8931-85a5b1934625" />



- **Bar Plot of Team Ratings by Playoff Status**: It revealed that teams with higher ratings had a higher likelihood of qualifying for the playoffs.

```r
attendance_joined %>%
  filter(!is.na(weekly_attendance)) %>%
  ggplot(aes(team_name, simple_rating, fill = playoffs)) +
  geom_bar(stat = "identity", position = "dodge") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  coord_flip()
```

<img width="508" alt="Screenshot 2025-02-07 at 6 49 36 PM" src="https://github.com/user-attachments/assets/333114ee-c051-481f-a0b2-614161d8e181" />


- **Weekly Attendance Trend**: A bar plot illustrated the fluctuation of weekly attendance throughout the season.

```r
attendance_joined %>%
  ggplot(aes(factor(week), weekly_attendance, fill = playoffs)) +
  geom_bar(stat = "identity") +
  labs(title = "Weekly attendance throughout the years", x = "Week of the year", y = "Weekly Attendance") +
  theme_light()
```

<img width="486" alt="Screenshot 2025-02-07 at 6 49 44 PM" src="https://github.com/user-attachments/assets/f68f885c-39ea-41ee-8d6f-1c54aeb2e4bd" />


#### 3. Model Building

To predict weekly attendance, two models were built:

- **Linear Regression Model**: This model was trained to predict weekly attendance based on several features such as team rating and strength of schedule.

```r
linear_model = linear_reg() %>%
  fit(data = train_data, weekly_attendance ~ .)
```

- **Random Forest Model**: A random forest regression model was also created to predict weekly attendance.

```r
rf_spec <- rand_forest(mode = "regression") %>%
  set_engine("ranger")

rf_model = rf_spec %>%
  fit(weekly_attendance ~ ., train_data)
```

#### 4. Model Evaluation

The models were evaluated using **Root Mean Squared Error (RMSE)** on both training and test datasets. 

- The **random forest model** had a lower RMSE on the training data, but its performance decreased on the test data, indicating overfitting.
- The **linear regression model** showed consistent performance across both training and test data, suggesting it generalized well.

```r
# Evaluating performance using RMSE
results_train %>%
  group_by(method) %>%
  rmse(truth = real_value, estimate = .pred)

results_test %>%
  group_by(method) %>%
  rmse(truth = real_value, estimate = .pred)
```

#### 5. Cross-Validation and Model Refinement

Cross-validation was applied to improve the random forest model’s generalizability. The model was re-trained using resampling techniques to get a more robust evaluation.

```r
rf_res <- fit_resamples(
  rf_spec,
  weekly_attendance ~ .,
  nfl_folds,
  control = control_resamples(save_pred = TRUE)
)

rf_res %>%
  collect_metrics()
```

#### 6. Final Model Comparison

After retraining, the random forest model's performance was closer to that of the linear regression model on the test data.

```r
new_results_test %>%
  group_by(method) %>%
  rmse(truth = real, estimate = .pred)
```

#### Conclusion

- **Data Analytical Skills**: This project demonstrated various data analytical skills including data cleaning, feature selection, visualization, model training, evaluation, and improvement.
- **Key Insights**: The models provided insights into how factors like team rating, strength of schedule, and playoff qualification relate to weekly attendance.
- **Model Performance**: The linear regression model performed consistently well, while the random forest model exhibited signs of overfitting. By applying cross-validation, the performance gap between the models was reduced.

This project not only showcases the application of data science tools but also emphasizes the importance of model evaluation and refinement in building effective predictive models.
