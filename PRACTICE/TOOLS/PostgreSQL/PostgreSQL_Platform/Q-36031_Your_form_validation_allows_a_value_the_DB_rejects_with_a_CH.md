---
type: "practice"
difficulty: "Intermediate"
---

# Your form validation allows a value the DB rejects with a CHECK/unique violation at 3am; how do you surface a clean error to the user and keep DB constraints as the source of truth?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When a form validation allows an invalid value that fails a PostgreSQL CHECK constraint, the database will reject it at 3am or any time. To surface a clean error to the user while maintaining the database constraints as the source of truth, we need to handle this situation gracefully in our application code. This involves catching the specific exception from the database and converting it into an understandable message for the user.

#### Triage

The question is asking how to manage errors when a form submission violates a CHECK constraint or uniqueness constraint in PostgreSQL, ensuring that the user receives clear feedback while maintaining the integrity of the database constraints. We will assume that our application uses a common ORM like Sequelize or direct SQL queries via Node.js/Python etc.

#### First Principles Solution
Database Constraint: Ensure your unique or check constraint is properly defined in PostgreSQL.
Application Code: Handle specific exceptions from the database to provide user-friendly error messages.
Error Handling Layer: Implement a middleware layer that catches these errors and translates them into human-readable messages.
Step-by-Step Solution:

Define the Constraint:

#### Example
constraint: ALTER TABLE users ADD CONSTRAINT unique_email CHECK (email IS NOT NULL AND email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' );

Database Interaction with Node.js/Python:

#### Example
using Node.js and Sequelize:

JAVASCRIPT
try {
    await User.create({ email: 'invalid-email' });
} catch (error) {
    if (error.code === '23505') { // Unique violation code
        res.status(400).json({ message: "Email already exists. Please choose another one." });
    } else {
        throw error; // Rethrow unexpected errors
    }
}

#### Example
using Python with SQLAlchemy:

PYTHON
try:
    user = User(email='invalid-email')
    db.session.add(user)
    db.session.commit()
except IntegrityError as e:
    if 'unique_email' in str(e):
        return {"message": "Email already exists. Please choose another one."}, 400
    else:
        raise e

Error Handling Middleware:

Implement a global error handling middleware to catch and format database errors.
JAVASCRIPT
app.use((err, req, res, next) => {
    if (err.name === 'SequelizeValidationError') {
        return res.status(400).json({
            message: err.errors.map(error => error.message)
        });
    }
    if (err.code && err.code === '23505') { // Unique violation
        return res.status(400).json({ message: "Email already exists. Please choose another one." });
    }
    next(err);
});

#### Closing Summary

By implementing specific error handling in your application code, you can catch database constraints violations and provide clean, user-friendly error messages. This ensures that the database remains the single source of truth for data integrity while improving the user experience.

#### Common Mistakes/Follow-Ups
Ignoring Database Errors: Not properly catching and handling database exceptions can lead to unexpected behavior.
Inconsistent Error Messages: Providing inconsistent or generic error messages can confuse users.
Neglecting Logging: Not logging database errors for further analysis can hinder troubleshooting.
Failing to Validate in Both Layers: Relying solely on the client-side validation without database constraints can lead to security vulnerabilities.

