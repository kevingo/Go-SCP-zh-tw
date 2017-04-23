輸入資料驗證
===============

在網路應用安全上，如果我們對於使用者的輸入資料不加以驗證，那會是很危險的。我們透過 "輸入資料驗證" 和 "輸入資料清理" 來加以解決這個問題。這些驗證應該在應用程式的每一個環節中進行，同時，驗證的程序必須在可信任的系統上執行(例如：在伺服器端)。

As noted in the [OWASP SCP Quick Reference Guide][1], there are sixteen
bullet points that cover the issues that the developer should be aware of when
dealing with Input Validation.
A lack of consideration for these security risks when developing an application
is one of the main reasons [Injection][2] ranks as the number 1 vulnerability
in the "[OWASP Top 10][3]".

User interaction is a staple of the current development paradigm in web
applications. As web applications become increasingly richer in content and
possibilities, user interaction and submitted user data also increases.
It is in this context that Input Validation plays a significant role.

When applications handle user data, submitted data **must be considered
insecure by default**, and only accepted after the appropriate security checks
have been made. Data sources must also be identified as trusted, or untrusted,
and in case of an untrusted source, validation checks must be made.

In this section an overview of each technique is provided, along with a sample
in Go to illustrate the issues.

* Validation
    1. User Interactivity
        * Whitelisting
        * Boundary checking
        * Character escaping
        * Numeric validation
    2. File Manipulation
    3. Data sources
        * Cross-system consistency checks
        * Hash totals
        * Referential integrity
        * Uniqueness check
        * Table look up check
* Post-validation Actions
    1. Enforcement Actions
        * Advisory Action
        * Verification Action
* Sanitization
    1. Check for invalid UTF-8
        * Convert single less-than characters (<) to entity
        * Strip all tags
        * Remove line breaks, tabs and extra white space
        * Strip octets
        * URL request path

[1]: https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf
[2]: https://www.owasp.org/index.php/Top_10_2013-A1-Injection
[3]: https://www.owasp.org/index.php/Top_10_2013-Top_10
