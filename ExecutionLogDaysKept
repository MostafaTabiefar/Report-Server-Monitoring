# Changing the `ExecutionLogDaysKept` Setting in Report Server

By default, a **Report Server** instance retains execution logs for the most recent **60 days**, which equates to about **two months** of data. If you need to adjust this retention period, consider the following options:

- **Set the value to `0`**: Disables data retention entirely (no logs are kept).
- **Set the value to `-1`**: Retains logs **indefinitely**.

For this dashboard, it is recommended to retain execution log data for at least **one year (366 days)**. However, please be aware that retaining logs for long periods without proper management can result in several potential issues.

---

## Potential Issues with Extended Retention

### 1. **Disk Space Usage**
- **Problem**: Logs can accumulate quickly, especially if there is a high volume of reports or verbose logging enabled.
- **Impact**: This can result in excessive disk space usage, potentially impacting the **performance** of the report server and other applications that share the same disk space.

### 2. **Difficulty in Maintenance**
- **Problem**: Managing a large number of logs becomes more challenging over time.
- **Impact**: Searching through logs to identify specific issues or patterns can become **time-consuming** and inefficient.

### 3. **Security Risks**
- **Problem**: Retaining logs indefinitely increases the risk of **exposing sensitive data** in the event of a breach.
- **Impact**: Logs may contain sensitive information, such as user details, reports, or configurations, that could be exploited by attackers.

---

## How to Change the `ExecutionLogDaysKept` Setting

To modify the log retention period, execute the following SQL query after connecting to your Report Server instance:

```sql
UPDATE dbo.ConfigurationInfo
SET Value = '366'  -- Or your desired retention period
WHERE Name = 'ExecutionLogDaysKept';