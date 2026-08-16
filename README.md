# Machine-Learning-through-Django
# Django Iris Flower Prediction & CSV Analysis Project

## 1. Project Overview

This project is a **Django-based Iris Flower Prediction and CSV Analysis
application**.

The application provides two major features:

1.  **CSV File Upload and Analysis**
    -   Upload an Iris dataset in CSV format.
    -   Read the uploaded CSV using Pandas.
    -   Identify the target/category column automatically as the last
        column.
    -   Display available flower categories.
    -   Filter records according to the selected category.
    -   Display filtered records in a table.
    -   Provide pagination with 10 records per page.
2.  **Iris Flower Prediction**
    -   Accept Sepal Length, Sepal Width, Petal Length, and Petal Width
        as input.
    -   Validate the input values using a Django Form.
    -   Train a Decision Tree Classifier using the uploaded dataset.
    -   Save the trained model as `model.pkl`.
    -   Reuse the saved model for subsequent predictions.
    -   Return the predicted Iris species through AJAX.

------------------------------------------------------------------------

## 2. Technologies Used

  Technology                 Purpose
  -------------------------- -------------------------------------
  Python                     Core programming language
  Django                     Web application framework
  Pandas                     CSV reading and data processing
  Scikit-learn               Machine learning model
  Decision Tree Classifier   Iris species prediction
  Pickle                     Saving/loading the trained ML model
  SQLite                     Django default database
  HTML5                      Web page structure
  Bootstrap 5                User interface styling
  JavaScript                 Client-side interaction
  jQuery                     AJAX requests and DOM manipulation
  CSV                        Dataset input format

------------------------------------------------------------------------

## 3. Project Structure

``` text
decision_tree/
│
├── manage.py
├── db.sqlite3
│
├── csv_upload/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── forms.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views.py
│
├── pandas_fileUpload/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── ml/
│   ├── IRIS.csv
│   ├── model.pkl
│   └── train_model.py
│
├── media/
│   ├── IRIS.csv
│   ├── iris_sample.csv
│   └── temp_iris.csv
│
├── templates/
│   └── csv_uplaod/
│       └── csv_upload.html
│
└── static/
    └── js/
        ├── jquery.min.js
        └── script.js
```

> Note: The template directory is named `csv_uplaod` in the current
> project. This spelling is used consistently by the existing Django
> view and template configuration.

------------------------------------------------------------------------

# 4. Application Flow

The overall application flow is:

``` text
User opens Django application
          |
          v
CSV Upload Page
          |
          v
Upload CSV File
          |
          v
Django saves file in media/
          |
          v
Pandas reads CSV
          |
          v
Last column is treated as target/category
          |
          v
Categories returned through JSON
          |
          v
User selects a category
          |
          v
User clicks "Analyze Report"
          |
          v
Records are filtered
          |
          v
Pagination is applied
          |
          v
Filtered records displayed in table

          AND

User enters flower measurements
          |
          v
Django Form validates values
          |
          v
Decision Tree model is loaded
          |
          +---- model.pkl exists ----> Load model
          |
          +---- model.pkl missing ---> Train model
                                      |
                                      v
                                  Save model.pkl
          |
          v
Predict Iris species
          |
          v
Return prediction using JSON
          |
          v
Display prediction on webpage
```

------------------------------------------------------------------------

# 5. Django Configuration

The Django project is named:

``` text
pandas_fileUpload
```

The application is:

``` text
csv_upload
```

The root URL configuration contains:

``` python
urlpatterns = [
    path("admin/", admin.site.urls),
    path("", views.home, name="home"),
    path("csv-upload/", include("csv_upload.urls"))
]
```

Therefore:

  URL                      Purpose
  ------------------------ -------------------------------------
  `/`                      Main CSV upload and prediction page
  `/csv-upload/`           CSV upload endpoint
  `/csv-upload/analyze/`   Dataset analysis endpoint
  `/csv-upload/predict/`   Flower prediction endpoint
  `/admin/`                Django administration

------------------------------------------------------------------------

# 6. CSV Upload Functionality

The CSV upload is handled by the `file_upload()` view.

The user selects a CSV file from the browser.

The JavaScript sends the file using an AJAX request:

``` javascript
$.ajax({
    url: "/csv-upload/",
    method: "POST",
    data: formData,
    processData: false,
    contentType: false
});
```

Django receives the uploaded file using:

``` python
upload_file = request.FILES["csv_file"]
```

The file is then saved inside the media directory.

``` python
save_path = os.path.join(settings.MEDIA_ROOT, upload_file.name)
```

The uploaded file is written in chunks:

``` python
with open(save_path, mode="wb+") as dest:
    for chunk in upload_file.chunks():
        dest.write(chunk)
```

This allows Django to process uploaded files without requiring the
entire file to be loaded into memory at once.

------------------------------------------------------------------------

# 7. Session-Based File Tracking

After uploading the CSV file, the application stores its path in the
user's Django session:

``` python
request.session["csv_path"] = save_path
```

This allows the prediction functionality to identify the currently
uploaded dataset.

The session is also useful because the prediction request is sent
separately from the upload request.

------------------------------------------------------------------------

# 8. Automatic Category Detection

After the CSV file is uploaded, Pandas reads the file:

``` python
df = pd.read_csv(save_path)
```

The application assumes that the **last column is the target column**:

``` python
target_column = df.columns[-1]
```

The unique categories are then extracted:

``` python
category = df[target_column].dropna().unique().tolist()
```

For the standard Iris dataset, these categories are normally:

``` text
Iris-setosa
Iris-versicolor
Iris-virginica
```

The categories are returned to the browser as JSON.

Example response:

``` json
{
    "status": true,
    "category": [
        "Iris-setosa",
        "Iris-versicolor",
        "Iris-virginica"
    ],
    "file_name": "IRIS.csv",
    "message": "File uploaded successfully"
}
```

------------------------------------------------------------------------

# 9. Report Analysis

The `analyze_report()` view performs category-based filtering.

The selected category is received from the AJAX request:

``` python
select_category = request.POST.get("category")
```

The uploaded file name is also received:

``` python
file_name = request.POST.get("file_name")
```

Pandas loads the CSV:

``` python
df = pd.read_csv(file_path)
```

The target column is again identified as the final column:

``` python
target_column = df.columns[-1]
```

The dataset is filtered:

``` python
filtered_df = df[df[target_column] == select_category]
```

The target column is removed before displaying the table:

``` python
filtered_df = filtered_df.drop(columns=[target_column])
```

------------------------------------------------------------------------

# 10. Column Name Formatting

The project includes a column mapping so that technical CSV column names
can be displayed in a more readable format.

Example:

``` python
column_mapping = {
    "sepal_length": "Sepal Length",
    "sepal_width": "Sepal Width",
    "petal_length": "Petal Length",
    "petal_width": "Petal Width",
    "SepalLengthCm": "Sepal Length",
    "SepalWidthCm": "Sepal Width",
    "PetalLengthCm": "Petal Length",
    "PetalWidthCm": "Petal Width",
    "Id": "Id",
    "id": "Id"
}
```

This allows datasets using slightly different Iris column naming
conventions to be displayed consistently.

------------------------------------------------------------------------

# 11. Pagination

The application uses Django's `Paginator`.

The number of records per page is:

``` python
paginator = Paginator(records, 10)
```

Therefore, the report displays a maximum of **10 records per page**.

The response includes:

``` python
"current_page": page_obj.number,
"total_pages": paginator.num_pages,
"has_next": page_obj.has_next(),
"has_previous": page_obj.has_previous()
```

The JavaScript uses these values to create:

-   Previous button
-   Current page indicator
-   Next button

Example:

``` text
← Previous     Page 2 of 5     Next →
```

------------------------------------------------------------------------

# 12. Machine Learning Component

The machine learning component uses the **Decision Tree Classifier**
from Scikit-learn.

The training code is located in:

``` text
ml/train_model.py
```

The model is imported into the Django view:

``` python
from ml.train_model import train_model
```

------------------------------------------------------------------------

# 13. Dataset Preparation

The training function reads the CSV using Pandas:

``` python
df = pd.read_csv(csv_path)
```

The target variable is assumed to be the last column:

``` python
target_column = df.columns[-1]
y = df[target_column]
```

The feature dataset is created by removing the target column:

``` python
X = df.drop(target_column, axis=1)
```

------------------------------------------------------------------------

# 14. ID Column Handling

Some Iris datasets contain an `Id` column.

The project removes this column because it is an identifier rather than
a meaningful flower measurement.

``` python
for id_col in ["Id", "id"]:
    if id_col in X.columns:
        X = X.drop(id_col, axis=1)
```

The final machine learning features are therefore the flower
measurements:

``` text
Sepal Length
Sepal Width
Petal Length
Petal Width
```

------------------------------------------------------------------------

# 15. Train-Test Split

The dataset is divided into training and testing data:

``` python
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42
)
```

The configuration means:

-   80% of the data is used for training.
-   20% is used for testing.
-   `random_state=42` provides reproducible splitting.

------------------------------------------------------------------------

# 16. Decision Tree Model

The classifier is created using:

``` python
model = DecisionTreeClassifier(random_state=42)
```

The model is trained using:

``` python
model.fit(X_train, y_train)
```

The trained model then predicts the test dataset:

``` python
y_pred = model.predict(X_test)
```

------------------------------------------------------------------------

# 17. Model Accuracy

The project calculates accuracy using:

``` python
accuracy = accuracy_score(y_test, y_pred)
```

The result is printed:

``` python
print("Accuracy :", accuracy)
```

Accuracy measures the proportion of correctly classified test samples.

------------------------------------------------------------------------

# 18. Model Serialization

After training, the Decision Tree model is stored using Python's
`pickle` module:

``` python
with open("ml/model.pkl", "wb") as file:
    pickle.dump(model, file)
```

The saved file is:

``` text
ml/model.pkl
```

This prevents the application from having to train the model every time
a prediction is requested.

------------------------------------------------------------------------

# 19. Automatic Model Retraining

When a new CSV file is uploaded, the existing model is deleted:

``` python
model_path = os.path.join(settings.BASE_DIR, "ml", "model.pkl")

if os.path.exists(model_path):
    os.remove(model_path)
```

This is important because a newly uploaded dataset may contain different
training data.

The next prediction request checks whether the model exists:

``` python
if not os.path.exists(model_path):
    model = train_model(csv_path)
```

If the model already exists, it is loaded:

``` python
with open(model_path, "rb") as file:
    model = pickle.load(file)
```

Therefore, the application follows this logic:

``` text
New CSV uploaded
      |
      v
Delete old model.pkl
      |
      v
First prediction request
      |
      v
Train new Decision Tree
      |
      v
Save model.pkl
      |
      v
Make prediction
```

For later predictions:

``` text
Prediction request
      |
      v
model.pkl exists
      |
      v
Load saved model
      |
      v
Make prediction
```

------------------------------------------------------------------------

# 20. Prediction Form

The Django form is defined in:

``` text
csv_upload/forms.py
```

The form contains four fields:

``` python
class PredictionForm(forms.Form):

    sepal_length = forms.FloatField()
    sepal_width = forms.FloatField()
    petal_length = forms.FloatField()
    petal_width = forms.FloatField()
```

These values represent the four measurements required by the Iris
classifier.

------------------------------------------------------------------------

# 21. Input Validation

The form validates each input against the expected Iris dataset ranges.

### Sepal Length

``` text
4.3 to 7.9
```

### Sepal Width

``` text
2.0 to 4.4
```

### Petal Length

``` text
1.0 to 6.9
```

### Petal Width

``` text
0.1 to 2.5
```

For example:

``` python
def clean_sepal_length(self):
    value = self.cleaned_data["sepal_length"]

    if value < 4.3 or value > 7.9:
        raise forms.ValidationError(
            "Sepal Length must be between 4.3 and 7.9."
        )

    return value
```

This prevents obviously invalid values from being sent to the machine
learning model.

------------------------------------------------------------------------

# 22. Prediction Process

The prediction request is handled by:

``` python
def predict_flower(request):
```

The form receives the submitted data:

``` python
form = PredictionForm(request.POST)
```

If validation succeeds, the model is loaded or trained.

The four input values are passed to the model:

``` python
prediction = model.predict([[
    form.cleaned_data["sepal_length"],
    form.cleaned_data["sepal_width"],
    form.cleaned_data["petal_length"],
    form.cleaned_data["petal_width"]
]])
```

The prediction is returned as JSON:

``` python
return JsonResponse({
    "status": True,
    "prediction": prediction[0]
})
```

------------------------------------------------------------------------

# 23. AJAX Integration

The frontend uses jQuery AJAX instead of traditional full-page form
submissions.

Three major AJAX operations are implemented:

### Upload CSV

``` text
POST /csv-upload/
```

### Analyze Report

``` text
POST /csv-upload/analyze/
```

### Predict Flower

``` text
POST /csv-upload/predict/
```

This approach allows the page to update specific sections without
reloading the entire webpage.

------------------------------------------------------------------------

# 24. Frontend Interface

The main page contains two major cards.

## CSV File Upload

The first card contains:

-   CSV file selector
-   Upload button
-   Category dropdown
-   Analyze Report button
-   Upload/report status message

## Predict Flower

The second card contains:

-   Sepal Length
-   Sepal Width
-   Petal Length
-   Petal Width
-   Predict button
-   Prediction result

The report table and pagination controls are displayed below the cards.

------------------------------------------------------------------------

# 25. Bootstrap

Bootstrap 5.2.3 is used for styling.

The project uses components such as:

``` text
container
row
col-lg-6
card
form-control
form-select
btn
alert
table
badge
```

This gives the application a responsive interface without requiring a
large custom CSS file.

------------------------------------------------------------------------

# 26. CSRF Protection

Django CSRF protection is enabled through:

``` python
"django.middleware.csrf.CsrfViewMiddleware"
```

The HTML form includes:

``` django
{% csrf_token %}
```

The AJAX requests retrieve the generated token:

``` javascript
$("input[name=csrfmiddlewaretoken]").val()
```

and send it with the request.

This helps protect POST requests against Cross-Site Request Forgery
attacks.

------------------------------------------------------------------------

# 27. Complete User Workflow

### Step 1: Open the application

The user visits:

``` text
/
```

### Step 2: Upload CSV

The user selects an Iris CSV file and clicks:

``` text
Upload
```

### Step 3: Dataset Processing

Django:

1.  Receives the file.
2.  Saves it to `media/`.
3.  Stores the path in the session.
4.  Deletes the previous ML model.
5.  Reads the CSV using Pandas.
6.  Identifies the last column as the target.
7.  Extracts unique categories.

### Step 4: Select Category

The category dropdown becomes enabled.

The user selects a category such as:

``` text
Iris-setosa
```

### Step 5: Analyze Report

The user clicks:

``` text
Analyze Report
```

The selected category is sent to Django.

The server filters the dataset and returns the records.

### Step 6: Pagination

The records are displayed 10 at a time.

### Step 7: Enter Flower Measurements

The user enters:

``` text
Sepal Length
Sepal Width
Petal Length
Petal Width
```

### Step 8: Validate Inputs

Django checks whether each measurement is within the allowed range.

### Step 9: Train or Load Model

If `model.pkl` does not exist:

``` text
Train Decision Tree
```

Otherwise:

``` text
Load model.pkl
```

### Step 10: Prediction

The model predicts the Iris species.

### Step 11: Display Result

The predicted species is displayed in the prediction result area.

------------------------------------------------------------------------

# 28. Example Prediction

Suppose the user enters:

``` text
Sepal Length : 5.1
Sepal Width  : 3.5
Petal Length : 1.4
Petal Width  : 0.2
```

The application sends these values to the Decision Tree model.

A typical result for these measurements is:

``` text
Predicted Species: Iris-setosa
```

The exact prediction depends on the model trained from the uploaded
dataset.

------------------------------------------------------------------------

# 29. Error Handling

The application handles several types of invalid requests.

### No CSV File

If the user submits without a file:

``` json
{
    "status": false,
    "message": "No file uploaded"
}
```

### Invalid Request

For an unsupported request method:

``` json
{
    "status": false,
    "message": "Invalid Request"
}
```

### Invalid Prediction Input

If the prediction form is invalid, Django returns the form errors:

``` python
return JsonResponse({
    "status": False,
    "errors": form.errors
})
```

The JavaScript then displays the validation message next to the relevant
field.

------------------------------------------------------------------------

# 30. Important Design Decisions

## Automatic Target Detection

Instead of hard-coding a target column name, the application uses:

``` python
target_column = df.columns[-1]
```

This makes the application compatible with CSV files where the target
column has different names, provided the target is the final column.

## Automatic ID Removal

The application checks for both:

``` text
Id
id
```

and removes them from the machine learning feature set.

## Model Reuse

The trained model is serialized with Pickle so that repeated predictions
do not require retraining.

## Dataset-Specific Retraining

Uploading a new dataset deletes the existing model, ensuring that the
next prediction uses a model trained from the new dataset.

------------------------------------------------------------------------

# 31. How to Run the Project

Open a terminal and navigate to the project directory:

``` bash
cd decision_tree
```

Create and activate a virtual environment if required:

``` bash
python -m venv venv
```

Windows:

``` bash
venv\Scripts\activate
```

Install the required packages:

``` bash
pip install django pandas scikit-learn
```

Start the Django development server:

``` bash
python manage.py runserver
```

Open the application in a browser:

``` text
http://127.0.0.1:8000/
```

------------------------------------------------------------------------

# 32. Required Python Packages

The project requires at least:

``` text
Django
pandas
scikit-learn
```

The project also uses Python standard-library modules such as:

``` text
pickle
os
pathlib
```

jQuery and Bootstrap are used on the frontend.

------------------------------------------------------------------------

# 33. Sample Dataset Format

The project is designed around the Iris dataset.

A compatible CSV can contain columns similar to:

``` text
Id,SepalLengthCm,SepalWidthCm,PetalLengthCm,PetalWidthCm,Species
```

Example:

``` text
1,5.1,3.5,1.4,0.2,Iris-setosa
2,4.9,3.0,1.4,0.2,Iris-setosa
3,4.7,3.2,1.3,0.2,Iris-setosa
```

The application uses the last column as the target.

------------------------------------------------------------------------

# 34. Project Advantages

-   Simple Django-based machine learning deployment.
-   Supports CSV upload.
-   Uses Pandas for dataset processing.
-   Automatically detects the target column.
-   Supports category-based filtering.
-   Provides server-side pagination.
-   Uses a Decision Tree classifier.
-   Saves the trained model as a Pickle file.
-   Avoids unnecessary retraining.
-   Automatically retrains after a new CSV upload.
-   Provides Django form validation.
-   Uses AJAX for a smoother user experience.
-   Uses Bootstrap for responsive UI.

------------------------------------------------------------------------

# 35. Limitations

The current implementation also has some limitations:

1.  The application assumes the target column is the final CSV column.
2.  The prediction form assumes the dataset contains four Iris
    measurement features.
3.  The accepted prediction ranges are specifically based on the Iris
    dataset.
4.  Uploaded filenames are used when constructing the media path, so
    production deployments should add stronger filename/path validation.
5.  The Pickle model should only be loaded from trusted sources.
6.  `DEBUG = True` should not be used in production.
7.  `ALLOWED_HOSTS` must be configured for production deployment.
8.  The current project uses SQLite, which is suitable for development
    but may not be ideal for a production application.
9.  The machine learning training process occurs during a prediction
    request when a model does not yet exist.

------------------------------------------------------------------------

# 36. Possible Future Improvements

The project can be extended with:

-   Random Forest and Logistic Regression models.
-   Model accuracy displayed on the webpage.
-   Confusion matrix visualization.
-   Classification report.
-   ROC-AUC analysis where applicable.
-   Automatic feature detection.
-   Better dataset validation.
-   CSV schema validation.
-   File size restrictions.
-   Duplicate file handling.
-   Secure random upload filenames.
-   Background model training.
-   Model versioning.
-   Prediction history.
-   User authentication.
-   Database storage for prediction results.
-   Downloadable analysis reports.
-   Charts and visualizations using Matplotlib or Plotly.
-   REST API integration.
-   Docker deployment.
-   Production deployment using Gunicorn/Nginx or another suitable
    WSGI/ASGI setup.

------------------------------------------------------------------------

# 37. Conclusion

This project demonstrates how a **machine learning model can be
integrated into a Django web application**.

The application combines:

``` text
Django
   +
Pandas
   +
Scikit-learn
   +
Decision Tree
   +
Pickle
   +
AJAX/jQuery
   +
Bootstrap
```

The complete workflow allows a user to upload an Iris dataset, analyze
records by flower category, paginate the results, enter flower
measurements, and receive a machine learning prediction through a web
interface.

It is therefore a practical example of deploying a basic **Machine
Learning classification model inside a Django application**.
