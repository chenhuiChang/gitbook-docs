# 1.2. 基礎架構

在開始使用之前，你需要瞭解基本的PostgreSQL系統架構。認識PostgreSQL如何回應操作，有助於讓你更清楚瞭解以下的說明。

以資料庫的述語來說，PostgreSQL採用了主從式架構（client/server）。PostgreSQL會在進行下列操作時保持連線：

* 伺服器的執行緒，負責管理資料庫的檔案、受理用戶端的連線要求、執行相對應的資料庫動作。這樣的資料庫伺服端程式稱之為「postgres」。
* 用戶端的程式用來發起資料庫操作的行為，其設計的形態很廣泛：可能是文字介面的工具、圖型介面的程式、將資料庫內容顯示成網頁的網際網路伺服器、甚或是專用的資料庫管理工具。有一些用戶端程式是由PostgreSQL官方所提供，大部份由第三方的其他使用者所開發。

如同一般的主從式架構，用戶端與伺服端可以是兩台不同的主機，而他們透過TCP/IP的網路協定溝通。你應該將這個觀念謹記在心，因為某些在用戶端可以被存取的檔案，在伺服端可能就無法存取（或使用不同的檔案名稱）。

PostgreSQL伺服器可以管理來自多個用戶端的同步連線。為了達到這樣的功能，它會自我複製（fork）成新的執行緒，一對一地處理每一個連線。這個部份進一步來說，用戶端和新的伺服器執行緒之間的溝通，並不需要原始的postgres執行緒介入。也就是說，主要的資料庫服務執行緒會持續等待其他用戶端的連線，協助安排好其與伺服端執行緒的配對之後便完全交接，再回到等待的狀態。（當然，使用者完全不會察覺這些行為，在此說明僅僅是為了整體性的概念描繪）

