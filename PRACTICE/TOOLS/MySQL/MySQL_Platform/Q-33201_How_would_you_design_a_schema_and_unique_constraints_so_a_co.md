---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema and unique constraints so a coupon can be redeemed at most once per user and at most N times globally, race-safe?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to ensure that each coupon can only be redeemed by a user once, and globally no more than (N) times. This requires careful schema design, unique constraints, and handling of concurrent requests in a race-safe manner. Let's break down the solution step-by-step.

Firstly, the question is probing our understanding of database concurrency, unique constraints, and how to manage global counters efficiently without risking race conditions. We will need to use a combination of a coupon table with appropriate indexes and a transactional approach to ensure atomicity.

**Code:**

```sql
-- Coupon Table Schema
CREATE TABLE coupons (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(255) NOT NULL UNIQUE,  -- Unique coupon code
    user_id BIGINT,                     -- User who redeemed the coupon (nullable for global count)
    redemption_count INT DEFAULT 0,     -- Count of redemptions by this user
    global_redemption_count INT DEFAULT 0,  -- Global redemption count
    last_redemption TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- Timestamp of last redemption
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Example input / output:**

```

To ensure a coupon can be redeemed by the same user once and globally no more than (N) times, we'll use a transactional approach:

SQL
Copy
BEGIN TRANSACTION;

-- Step 1: Check if global count is within limit
SELECT @global_count := global_redemption_count FROM coupons WHERE code = 'COUPONABC';

IF @global_count < 5 THEN
    -- Step 2: Increment the global redemption count
    UPDATE coupons SET global_redemption_count = global_redemption_count + 1 WHERE code = 'COUPONABC';

    -- Step 3: Check if user has already redeemed this coupon before
    SELECT @user_count := redemption_count FROM coupons WHERE code = 'COUPONABC' AND user_id = 100;

    IF @user_count < 1 THEN
        -- Step 4: Increment the user's redemption count and set their ID in the coupon record
        UPDATE coupons SET redemption_count = redemption_count + 1, user_id = 100 WHERE code = 'COUPONABC';
    ELSE
        ROLLBACK;  -- User has already redeemed this coupon
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Coupon already redeemed by this user.';
    END IF;
ELSE
    ROLLBACK;  -- Global redemption limit exceeded
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Global redemption limit exceeded for this coupon.';
END IF;

COMMIT;
```

**Data:**

```json
{
  "coupon": {
    "id": 1,
    "code": "COUPONABC",
    "user_id": null,
    "redemption_count": 0,
    "global_redemption_count": 3,  // Example global redemption count for this coupon
    "last_redemption": "2023-10-05T14:48:00Z"
  }
}
```

