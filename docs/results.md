# Results and Conclusions

This section summarizes the outcomes of the lab, comparing the target's state before and after the mitigation was applied, and discusses the operational limitations of the current setup alongside potential future improvements.

---

## 1. Comparative Results

The following table illustrates the effectiveness of the Nginx mitigation rule applied to block the attacker (`10.10.10.20`):

| Test | Before Block | After Block |
| :--- | :--- | :--- |
| `GET /login/login.php` from `10.10.10.20` | `200 OK` | `403 Forbidden` |
| `POST /login/login.php` (valid credentials) from `10.10.10.20` | `200 OK` (successful login) | `403 Forbidden` |
| `GET /` from `10.10.10.20` | `200 OK` | `200 OK` (no changes) |
| Identifiable brute-force attempts in `access.log` | 1754 | 0 (blocked at Nginx layer) |
| Valid credential discovered by `ffuf` | `labadmin:Lab2024!` | N/A |

> **Conclusion:** The mitigation successfully blocked the attacker's IP from accessing the vulnerable login endpoint, preventing further authentication attempts while keeping the rest of the web application accessible.

---

## 2. Limitations

While the mitigation was effective for this specific lab scenario, it is important to recognize its limitations from a real-world defensive perspective. The primary objective of this lab was **Log Analysis**; therefore, the mitigation was applied retroactively based on manual log review, rather than being an active defense mechanism.

* **Manual and Static Blocking:** The IP was blocked only after the attack concluded and logs were analyzed. In a real-world scenario, a human attacker would have already completed the attack and potentially established persistence before any human intervention occurs.
* **IP-Based Blocking, Not Behavior-Based:** If the attacker rotates their IP (using a botnet, VPN, or proxy), the `deny 10.10.10.20;` rule becomes immediately useless.
* **No Rate Limiting:** There is no limit on requests per second/minute. The server allowed 1,754 attempts with zero friction until manual intervention was performed.
* **No Alerting:** The attack goes completely unnoticed until someone manually reviews the logs. There is no proactive notification system in place.
* **Log Rotation as an Operational Risk:** As observed during the analysis phase, logs rotate. If they are not centralized in time, evidence can be lost or scattered across multiple files—a significant problem in real production environments.
* **Permanent and Reversible:** The block remains permanently until manually removed (no TTL or automatic expiration), and the attacker can easily bypass it by simply changing their IP address.

---

## 3. Improvements

To transition this lab environment from a manual forensic exercise to a robust, actively defended system, the following improvements are recommended:

* **Fail2ban:** Automate pattern detection (e.g., >10 failures in 60 seconds) and apply blocks without manual intervention by reading directly from `access.log` or `lab_auth.log`.
* **Nginx Native Rate Limiting (`limit_req_zone`):** Limit the request rate to `/login/` per IP before the requests even become a problem.
* **Log Centralization:** Send logs to a remote syslog, a basic ELK stack, or Grafana Loki. This solves the log rotation issue and allows for historical searching.
* **Alerting:** Implement a simple script that sends a notification (email, webhook) when a threshold of failed attempts is exceeded.
* **CAPTCHA or Progressive Delay:** Introduce a CAPTCHA or progressive delays after $N$ failed attempts to increase the cost of the attack without fully blocking legitimate users.
* **Multi-Factor Authentication (MFA):** Implement MFA to drastically reduce the impact of an attack, even if the password is eventually compromised.