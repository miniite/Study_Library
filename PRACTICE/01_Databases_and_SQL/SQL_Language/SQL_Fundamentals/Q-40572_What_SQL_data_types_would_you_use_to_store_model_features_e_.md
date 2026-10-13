---
type: "practice"
difficulty: "Basic"
---

# What SQL data types would you use to store model features (e.g. INT vs FLOAT vs DECIMAL) and why does precision matter?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively store model features in a database, understanding the appropriate SQL data types is crucial. This decision influences storage efficiency, performance, and most importantly, the precision of numerical values, which can significantly impact model accuracy.

Let's first clarify what each data type represents:

INT: Stores integer numbers without decimal places.
FLOAT: Stores approximate numeric values with a fractional component but limited precision (typically 6 to 7 significant digits).
DECIMAL or NUMERIC: Stores exact numeric values, offering higher precision and control over the number of decimal places.

For storing model features, the choice between these types depends on whether the feature is an integer or a real number, as well as the required level of precision. For instance:

Model input features like counts (e.g., number of customers) would be stored using INT because they are always whole numbers.
Features representing real measurements (e.g., temperature in degrees Celsius) should use either FLOAT or DECIMAL. The choice between these depends on the importance of precision for your model.

Precision matters significantly because:

Loss of Information: Using a data type with insufficient precision can lead to information loss, affecting the accuracy of the model.
Storage Efficiency: More precise types like DECIMAL may use more space but are necessary when high accuracy is required.
Performance Impact: While FLOAT offers faster computation due to lower storage requirements, it might not be suitable for models requiring higher precision.

In summary, choosing between INT, FLOAT, and DECIMAL depends on the nature of the data and the model's sensitivity to numerical values. For fresher ML engineers, starting with INT or FLOAT is common but understanding when DECIMAL is needed can improve model performance in critical applications.

