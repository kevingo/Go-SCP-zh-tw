輸出編碼
===============

雖然在 [OWASP SCP Quick Reference Guide][1] 中針對輸出的編碼只有六個項目，但在網路應用程式中，輸出編碼的不正確做法是相當普遍的，這導致了一個最常見的漏洞：[Injection][2]。

當網站越來越複雜的同時，產生的資料來源也越多：使用者、資料庫、第三方服務等等。當你把這些資料來源都在瀏覽器上呈現，你沒有一個強大的輸出編碼策略時，injection 就會發生了。

當然，你已經知道我們會在本章節中處理這些安全問題，但你真的知道他們是如何發生，並且如何避免嗎？

[1]: https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf
[2]: https://www.owasp.org/index.php/Top_10_2013-A1-Injection
