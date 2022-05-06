# Web security

## Giới thiệu

a

## XSS attack

### ***Giới thiệu XSS attack***

Bình thường, các backend developer sẽ cố hết sức làm sao cho server của họ được an toàn và có độ bảo mật cao nhất có thể để tránh việc server bị hacker làm sập. Nhưng thua keo này ta bày keo khác, hacker thay vì nhắm tới server, thì họ sẽ quay sang nhắm tới browser của người dùng. Browser sẽ thực hiện mọi lệnh javascript mà nó có thể tìm thấy một cách không cảnh giác. Chính vì vậy, hacker có thể inject javascript code vào browser và chúng sẽ được thực hiện khi người dùng xem website, từ đó hacker có thể làm rất nhiều thứ bằng javascript trên browser của user, từ việc lấy cắp thông tin khi user nhập thông tin của họ, hay làm cho browser thực hiện một việc ngoài ý muốn của user (chẳng hạn như bình luận khắp mọi nơi mà user không hề biết). Việc inject javascript vào browser như vậy được gọi là Cross-site scripting (XSS) attack.

### ***Phân loại và phương thức***

#### **Stored Cross-Site Scripting Attacks**

Stored Cross-Site Scripting Attacks là một cách tấn công XSS mà kẻ tấn công sẽ inject javascript vào database (bằng cách để code javascript trong tag `<script>`), thông qua các input thông thường mà website cho phép nhập như bình luận, chỉnh sửa thông tin tài khoản,... Không phải để làm hại tới database, mà để cho trình duyệt thực hiện những dòng javascript đó mỗi lần website được truy cập.
Ví dụ với một website cho người dùng thích làm bánh tên là Breddit, website cho phép người dùng tạo bài đăng và bình luận, và bài đăng, bình luận nào cũng sẽ được lưu vào database để hiện ra cho người khác khi họ xem website.

![Hacker đang chèn mã javascript độc vào phần comment](images/Breddit1.png 'Hacker đang chèn mã javascript độc vào phần comment')

Nếu như người làm ra website không hề cài đặt biện pháp phòng chống XSS nào, thì một kẻ tấn công có thể tạo một bình luận chứa mã javascript, thì từ đó những người nào truy cập website mà thấy bình luận đó thì mã javascript trong đấy cũng sẽ được thực hiện.

![Mã javascript độc của hacker đã được thực hiện](images/Breddit2.png 'Mã javascript độc của hacker đã được thực hiện')

Đây chỉ là một lệnh alert vô hại, chỉ gây phiền toái cho những user khác. Nhưng đây cũng là cách mà hacker biết rằng một website có thực hiện XSS được hay không, và thế là từ đó hành động của họ sẽ nguy hiểm hơn.

#### **Reflected Cross-Site Scripting Attacks**

Thay vì inject javascript vào database để chúng được thực hiện mỗi lần 1 browser truy cập website, thì hacker còn có thể inject javascript vào trong HTTP Request. Vì nếu như website bị tấn công có lấy một phần trong HTTP Request để show nó trong trang web (thường là query tìm kiếm trong URL) thì đoạn javascript của hacker hoàn toàn có thể được browser thực hiện và thực hiện các mục đích nguy hiểm, đó gọi là Reflected Cross-Site Scripting Attacks, vì server đã reflect (phản chiếu) lại những gì hacker gửi.

Ví dụ một website có cho người dùng tìm kiếm đồ uống bằng url có query như sau:
<https://mywebsite.com/search?product=coffee>
Và website sẽ hiển thị lại thông báo kết quả cho người dùng như sau trong HTML:\
`<p>Kết quả tìm kiếm cho coffee</p>`

Giả sử người làm ra website này không có biện pháp phòng chống nào cho reflected XSS, thì kẻ tấn công có thể nhập url như sau:\
`https://mywebsite.com/search?product>=<script>alert(“hehe”);</script>`

Và website sẽ hiển thị trong HTML:\
`<p>Kết quả tìm kiếm cho <script>alert(“hehe”);</script></p>`

Như vậy, ta thấy là hacker cũng có thể inject javascript vào HTTP Request, cụ thể trong trường hợp này là URL trong HTTP Request, để browser thực hiện các lệnh javascript ấy nhằm các mục đích nguy hiểm. Tuy nhiên để dạng tấn công này được hiệu quả thì hacker phải lừa được người dùng khác nhấn vào URL mà hacker đã xây dựng, thì cuộc tấn công mới được thực hiện.

#### **DOM-Based Cross-Site Scripting Attacks**

Hai cách tấn công trên đều có thông qua server (lưu vào database, hiển thị 1 phần nội dung trong HTTP request) mới có khả năng thực hiện được. Nhưng vì sự phát triển của các framework ở client hiện nay nên tấn công XSS có thể không cần thông qua server mà chỉ cần lợi dụng lỗ hổng ở phía client, cụ thể là các lỗ hổng ở các Document Object Model (DOM) của javascript. Sự lợi dụng này được gọi là DOM-based Cross-site scripting attack.

Cụ thể, khi DOM-based XSS xảy ra, thì một tập dữ liệu có mã độc chảy từ một source sang 1 sink, trong đó source là nơi mà người dùng có thể nhập data vào, thường là URL, và sink là một lời gọi các hàm nguy hiểm ,các hàm nguy hiểm đó là:

```javascript
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

Nhưng tuỳ vào sink và cách website sử dụng chúng, kẻ tấn công phải suy nghĩ cách để inject được mã độc vào browser. Ví dụ như khi website sử dụng document.write() mà data mà kẻ tấn công đang nhắm tới được để trong cặp dấu ngoặc kép, thì kẻ tấn công phải thêm “> trước khi họ ghi `<script>` để đóng cặp ngoặc và element đó. Hoặc đối với innerHTML, thì tag `<script>` sẽ không được chạy, cho nên kẻ tấn công sẽ phải dùng các tag khác như img hay iframe để thực hiện XSS attack. Ví dụ:\
`element.innerHTML='... <img src=1 onerror=alert(“hehe”)> ...'`

Một trong những cách lợi dụng phổ biến là nhắm vào URI Fragment của URL làm source. URI fragment chính là phần phía sau dấu “#” của URL, để scroll tới element có id đó khi web page được load. Các Single-page app (SPA), vì muốn tránh việc browser bị nháy trắng mỗi khi refresh page, nên họ để cả app vào trong 1 static URL và dùng URI Fragment để nhớ trạng thái lúc trước (vị trí) và load lại trạng thái đó mà không bị nháy. Kỹ thuật này cũng được sử dụng vào các ứng dụng web có infinite-scrolling.

Bình thường thì URI fragment sẽ không được gửi cho server, cho nên ta không thể bảo vệ DOM-based XSS bằng cách tăng cường bảo mật trên code của phía server, ngoài ra thì các developer cũng sẽ không thể nhìn server log để dò tìm thấy dấu vết cuộc tấn công nào, cho nên ta luôn cần phải xem xét kỹ lưỡng code bên phía client.

Theo lẽ thường thì fragment thường sẽ không được ghi ra thẳng nội dung của website, nhưng nó sẽ được ghi nếu developer bất cẩn không phân biệt giữa query và fragment và thế là fragment sẽ được ghi ra nội dung của website:

```html
<p id="a">
 <script>
  queryPosition = document.location.href.indexOf("product=");
  if(queryPosition >= 0){
   var paramValue = decodeURIComponent(document.location.href.substring(queryPosition+8));
  document.write(`Result:  ${decodeURIComponent(paramValue)}`);
 }
 </script>
</p>
```

Như vậy:
`https://mywebsite.com/search?product=<script>alert(“hehe”);</script>`

Hoặc:
`https://mywebsite.com/search#product=<script>alert(“hehe”);</script>`

Đều có thể làm hiện alert, nhưng hacker nếu được thì họ sẽ dùng cái bên dưới vì URI Fragment sẽ không được gửi lên cho server và được ghi vào log, làm cho các developer của website rất khó nhận ra lỗ hổng.

### ***Phòng chống***

#### **Escape dynamic content**

Cách đơn giản nhất để tránh browser thực hiện những đoạn javascript không mong muốn, chính là không xem những nội dung lấy từ database, HTTP request, fragment hay nhiều nguồn khác (dynamic content) là những đoạn HTML. Cách để làm được điều này đó chính là ta sẽ escape những kí tự điều khiển của HTML thành những entity encoding để cho chúng không có ý nghĩa gì đối với HTML mà chỉ là nội dung bình thường. Cách này có thể dùng đổi phòng chống cả 3 loại XSS.
Bảng các entity encoding ứng với các kí tự điều khiển của HTML:
![Bảng entity encoding](images/EntityEncoding.png 'Bảng entity encoding')

Ví dụ với reflected XSS:\
`https://mywebsite.com/search?product=<script>alert(“hehe”);</script>`

Thì nó sẽ ra:\
`<p>Kết quả tìm kiếm cho &lt;script&gt;alert(&quot;hehe&quot;)&lt;/script&gt;</p>`

Như vậy cuộc tấn công này đã bị vô hiệu hoá.

Vì XSS đã trở nên khá phổ biến nên các framework hiện đại đều đã có escape dynamic content một cách tự động.

#### **Content Security Policy**

Một cách khác để chống việc browser thực hiện những javascript được inject vào qua các loại XSS kể trên, đó là “nói” với browser của user rằng không nên thực hiện những dòng javascript ở các nguồn nào, cách để ta “nói” cho browser biết đó chính là sử dụng Content Security Policy (CSP)

Đối với XSS, thì kẻ tấn công chỉ có thể inject inline javascript, là những đoạn javascript mà được ghi thẳng giữa tag `<script>`, và tag `<script>` nhưng có thuộc tính src để dẫn đến file javascript ở chỗ khác. Hai điều này có thể tránh được bằng cách “nói” với browser là đừng chạy inline javascript và file javascript có nguồn ở bên ngoài. Ta có thể “nói” được điều đó với policy như sau:\
`Content-Security-Policy: script-src 'self'`

CSP có thể được đặt ở HTTP headers, hoặc ta có thể trong tag meta của file HTML như sau:\
`<meta http-equiv="Content-Security-Policy" content="script-src 'self'">`

Ngoài việc dùng để chặn việc chạy các đoạn javascript, thì người ta còn hay dùng CSP để tạo violation reports, để báo cho server biết và lưu lại vào log. Một lúc nào đó team developer sẽ mở log đó ra xem và tìm những page nào có lỗ hổng, thì họ sẽ vá cùng 1 lúc để không phải quá dựa dẫm vào CSP. Chúng ta có thể làm điều này bằng policy:\
`Content-Security-Policy-Report-Only: script-src 'self'; report-uri https://example.com/csr-reports`

Content security policy có thể dùng để chống cả 3 loại XSS, ngoài ra nó còn có thể chống được nhiều loại tấn công khác.

### ***So sánh***

Độ nguy hiểm của cả 3 loại XSS đều như nhau vì nó tùy thuộc vào những gì mà kẻ tấn công để vào đoạn javascript được inject vào browser.

| Stored | Reflected | DOM-based |
|--------|-----------|-----------|
|Inject javascript vào csdl -> có giao tiếp với server -> xuất hiện trong server log|Inject javascript vào HTTP request -> có giao tiếp với server -> xuất hiện trong server log|Không giao tiếp với server -> Không xuất hiện trong server log -> khó phát hiện hơn|
|Lưu vào database -> Mọi user truy cập đều trở thành nạn nhân -> Khả năng dẫn truyền cao|Phải lừa được người khác bấm vào URL được xây dựng -> khả năng dẫn truyền thấp|Phải lừa được người khác bấm vào URL được xây dựng -> khả năng dẫn truyền thấp

## XSS request forgery

### Khái niệm XSS request forgery

abcd

### Mục đích XSS request forgery

abcd

### Phương thức và phòng chống XSS request forgery

abcd

## SQL Injection

### Khái niệm SQL Injection

abcd

### Mục đích SQL Injection

abcd

### Phương thức và phòng chống SQL Injection

abcd

## Clickjacking

### ***Khái niệm Clickjacking***

HTML có cho phép các lập trình viên có thể hiện nội dung của trang web khác bằng tag iframe, từ đó cho phép nội dung của các trang web có thể hoà trộn lẫn nhau nhưng vẫn có thể quản lý tốt được vì javascript từ trang web trong iframe không được chạy trong trang web mà dùng tag iframe.

Nhưng các hacker luôn tìm cách lợi dụng mọi thứ có lợi với họ, và iframe là điều không ngoại lệ. Họ có thể dùng tag iframe để hiện trang web của người khác trong trang web của mình, nhưng để một nút tàng hình lên trên một nút khác để lừa người dùng bấm vào, khi người dùng bấm vào thì kẻ tấn công có thể chạy javascript hoặc dẫn browser tới một website khác và thực hiện vô vàn các điều nguy hiểm. Lúc đó kẻ tấn công đã hijack (chiếm đoạt) cú click của user, hoặc có thể gọi tắt là Clickjacking.

Mấu chốt của clickjacking là ta phải để url của web chúng ta gần giống với url của web bị tấn công, để khi user có typo trong việc nhập url thì vô tình họ sẽ vào web của chúng ta mà không hề biết, vì ta đã làm web của mình giống với web bị tấn công.

### ***Phương pháp***

Ví dụ ta muốn thực hiện tấn công một web <http://www.victim.com>, thì ta để

HTML:\
`<iframe src="http://www.victim.com/"></iframe>`

CSS:

```css
iframe {
 height: 100vh;
 width: 100vw;
 border: none;
 position: absolute;
 left: 0;
 top: 0;
}
```

Ta dùng đơn vị vw, vh (viewport width/height) cho width, height để cho iframe có thể chiếm 100% kích thước page, và sử dụng position là absolute để nó có thể giữ nguyên tại một toạ độ cụ thể (0, 0). Thì lúc này website của chúng ta sẽ giống y hệt web bị tấn công.

Tiếp theo là bước tạo một button tàng hình và để nó lên một button nào đó của trang web gốc, kích thước, các cạnh càng giống nhau càng tốt. CSS để làm button đó tàng hình có thể là như sau:

```css
color: transparent;
background-color: transparent;
```

Còn về vị trí, kích thước, thì ta có thể dùng các thuộc tính như `width, height, margin, padding, position, top, left, …` để xếp button đúng chỗ ta cần.
Sau cùng thì ta phải làm cho button đó có lợi cho chúng ta, ta có thể để một tag `anchor` bên trong nó và dẫn `href` tới một website khác của mình, hay ta có thể dùng attribute `onclick` và gọi một hàm javascript và từ đó ta có thể làm được rất nhiều thứ khác.

### ***Phòng chống Clickjacking***

#### **Frame busting**

Frame busting là một kĩ thuật chống 1 website khỏi bị hiển thị trong 1 frame, từ đó có thể phòng chống tấn công Clickjacking. Frame busting thường được cài đặt bằng cách sử dụng javascript để luôn làm website của mình luôn là window trên cùng của bất kỳ trang web nào mà nó được hiển thị (URL luôn là của chính nó).
Ta có thể làm điều trên như sau:

```javascript
if (window.top != window.self) {
 top.location = self.location;
}
```

Tuy nhiên kẻ tấn công có nhiều cách để vô hiệu hoá kỹ thuật này, một trong những cách đó là sử dụng thuộc tính sandbox:
`<iframe src="http://www.victim.com/" sandbox=""></iframe>`

Công dụng của thuộc tính sandbox đó chính là khai báo tính năng nào của website trong iframe nên được sử dụng, tính năng nào không, như khả năng download hay chạy script bằng cú pháp allow-... . Còn nếu để trống thì mọi tính năng đều bị vô hiệu hoá, trong đó có khả năng chạy script của website trong iframe. Xem thêm ở
<https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#attr-sandbox>

#### **X-Frame-Options**

Khi ta không thể chống Clickjacking từ phía frontend, ta chỉ có thể dựa vào backend - phía server để giải quyết, X-Frame-Options là một trong các cách giải quyết đó. X-Frame-Options (XFO) là một cách để nói cho browser rằng 1 trang web có nên được hiển thị bên trong một `<frame>, <iframe>, <embed>, <object>` hay không, từ đó có thể tránh được tấn công Clickjacking.

Ta chỉ có thể để XFO vào trong HTTP header, chứ không thể để trong tag meta của file HTML. Ví dụ với Apache, để phòng chống clickjacking ta để như sau:\
`Header set X-Frame-Options "DENY"`

“DENY” có nghĩa là trang web đó không thể được hiện trong bất kỳ frame nào, dù cho nó là của ai, ở đâu. Ngoài ra còn có options “SAMEORIGIN”, đó là chỉ được xuất hiện trong cùng nguồn gốc của website trong frame.

Xem thêm ở <https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options>

#### **Phòng chống Clickjacking với Content Security Policy**

Như đã có nhắc tới ở phần XSS attack, Content Security Policy (CSP) ngoài chống được XSS attack, còn có thể chống được nhiều loại tấn công khác, trong đó có Clickjacking. Để chống được Clickjacking bằng CSP, ta chỉ cần sử dụng directive frame-ancestors:
Content-Security-Policy: frame-ancestors ‘none’;

‘none’ có nghĩa là không thể có frame nào chứa được website của mình, dù có là từ đâu, ở đâu. Ngoài ra ta có thể dùng ‘self’, hay một website cụ thể để chỉ có thể để trong `<iframe>` của chính website đó, hay một website mà mình tin tưởng. Nhưng điều lý tưởng là đừng nên cho ai để website của mình vào 1 frame.

Xem thêm ở <https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors>
