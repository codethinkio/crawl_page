# Hướng dẫn crawl link một Page

Bài viết hướng dẫn crawl link một Page (URL) bất kỳ với mục đích hướng dẫn, học tập không khuyến khích các bạn đi crawl tài nguyên công sức và trí tuệ người khác bỏ ra để dùng cho mục đích cá nhân bản thân.

Crawl trong hướng dẫn là crawl tất cả các link, tức là url nằm trong thẻ <a> của trang. Tuỳ thuộc vào mục đích lấy thông tin mà các bạn có thể tuỳ biến thêm.

**Github:** [https://github.com/codethinkio/crawl_page](https://github.com/codethinkio/crawl_page)

**Yêu cầu:**

- Nodejs và các package sau: `cheerio`, `request-promise`, `request`.

**Logic:** Chúng ta sẽ sử dụng `request` package để gửi yêu cầu lấy hết nội dung html của trang nào đó, trong ví dụ là `www.phimmoi.net`sau đó sẽ sử dụng `cheerio ` package để kiểm soát và bóc tách các thành phần đã lấy được, tiếp theo lọc nội dung (ở đây là các link) cho đúng với yêu cầu và lưu data đó lại một file json để sau này chúng ta sử dụng tiếp.

**Ứng dụng:** Crawl dữ liệu thì ứng dụng nhiều lắm, tuỳ vào mục đích bạn sử dụng dữ liệu làm gì. Đơn giản có thể là crawl lấy tiêu đề, nội dung bài viết về để dùng cho website của mình. Hoặc lấy hết tất cả các link của một website nào đó để tạo sitemap. Hoặc lấy data về phân tích, vì không phải data của một service nào cũng có API hoặc sẵn sàng cung cấp cho chúng ta, nhưng nó đã public thì kiểu gì chúng ta cũng sẽ lấy về được. Rồi, bắt đầu thôi.

### I. Khởi tạo dự án.

- Mình thường đặt dự án nodejs trong thư mục `www/nodepark` vậy nên sẽ `cd www/nodepark` để khởi tạo một dự án nodejs tại đây.

- ```shell
  mkdir crawl_page
  cd crawl_page
  ```

- Khởi tạo dự án với npm

  ```shell
  npm init
  ```

- Bạn điền các thông tin khởi tạo vào, dạng thế này.



  ```json
  {
    "name": "crawl_page",

    "version": "1.0.0",

    "description": "This is application crawl all link a Page",

    "main": "index.js",

    "scripts": {

      "test": "echo \"Error: no test specified\" && exit 1"

    },

    "keywords": [

      "crawl",

      "nodejs"

    ],

    "author": "codethink.io",

    "license": "MIT"

  }
  ```


- Tiếp theo chúng ta sẽ khai báo các package sử dụng để `npm` kéo về `node_modules`

  ```shell
  npm install cheerio request-promise request --save
  ```

- Kéo về thành công sẽ cho kết quả như vầy.

- ```
  + request-promise@4.2.2
  + request@2.88.0
  + cheerio@1.0.0-rc.2
  added 71 packages from 100 contributors and audited 105 packages in 7.192s
  found 0 vulnerabilities
  ```



### II. Viết chức năng

1. **Liệt kê các chức năng:**

   - Lấy nội dung của một page: `getPageContent`
   - Lấy tất cả link của page đó: `getPageLink`
   - Lọc link đã get về để sau này còn sử dụng: `filterLink`
   - Xuất ra file json data để lưu trữ sau này dùng tiếp: `exportFile`

2. **index.js**

   - Tạo file `index.js` để bắt đầu viết các function tại đây.

   - Khai báo các package sử dụng

     ```javascript
     const fs = require('fs')
     const cheerio = require('cheerio')
     const request = require('request-promise')
     ```

   - Khai báo target cần crawl và địa chỉ xuất file data json sau khi get về.

     ```javascript
     var url_target = "http://www.phimmoi.net/"
     var file_export = "./exports/phimmoi_net.json"
     ```

   - Viết chức năng get html của trang `www.phimmoi.net`

     ```javascript
     // function get page content
     const getPageContent = (uri) => {
       const options = {
         uri,
         headers: {
           'User-Agent': 'Request-Promise'
         },
         transform: (body) => {
           return cheerio.load(body)
         }
       }

       return request(options)
     }
     ```

   - Như bạn thấy đoạn `return cheerio.load(body)` đã nhúng toạn bộ nội dung html vào `cheerio` chỉ một function đơn giản đã lấy về hết nội dung trang web rồi. Tiếp theo chúng ta sẽ lọc nội dung đó chỉ lấy ra các link thôi.

   - ```javascript
     // function get all link on page
     const getPageLink = (html) => {
       const $ = cheerio.load(html)
       var tags = $('a')
       var links = []
       $(tags).each( ( _index, link) => {
         links.push( $(link).attr('href') );
       });

       return links
     }
     ```

   - Function này mình lấy `html` đã lấy về cho vào `cheerio` sau đó load ra hết các thẻ <a>, tất nhiên sẽ có nhiều thẻ <a> nên chúng ta sẽ `.each` các thẻ ra để `get` link `push` vào một mảng tên là `links`

   - Đến đây có vẻ nhiệm vụ chính đã hoàn thành, nhưng không. Nếu bạn `console links` này ra sẽ thấy khá nhiều link lỗi và chúng ta không cần dùng đến, ví dụ: `underfine, login/xxx, register/xxx, javascript:void(0) ` Chúng ta cần lọc mảng này để có data đẹp và sexy hơn.

     ```javascript
     // function filter link beautiful
     const filterLink = (links) => {
       links.forEach((link, _index) => {

           // Function remove item has value "xyz"
           if( link == undefined ||
               link == "./" ||
               link == "/" ||
               link == "javascript:void(0)" ||
               link.split("/")[0] == "login" ||
               link.split("/")[0] == "logout" ||
               link.split("/")[0] == "register"
             ){
             delete links[_index]
             return
           }

           // Function regex value http https www http://www https://www
           var  _regex = new RegExp("https?:\/\/(?:www\.|(?!www))[a-zA-Z0-9][a-zA-Z0-9-]+[a-zA-Z0-9]\.[^\s]{2,}|www\.[a-zA-Z0-9][a-zA-Z0-9-]+[a-zA-Z0-9]\.[^\s]{2,}|https?:\/\/(?:www\.|(?!www))[a-zA-Z0-9]\.[^\s]{2,}|www\.[a-zA-Z0-9]\.[^\s]{2,}")

           if(!_regex.test(link)){
               links[_index] = url_target + link
           }

           return
       });

       var links = links.filter(function (link) {
         return link != null;
       });

       return links
     }
     ```

   - Chà đoạn code này dài, đọc không hiểu gì, lại có đoạn mã loằng ngoằng gì thế kia, bóc tách nó ra phân tích chút nhé.

   - Đoạn này,

   - ```javascript
     // Function remove item has value "xyz"
           if( link == undefined ||
               link == "./" ||
               link == "/" ||
               link == "javascript:void(0)" ||
               link.split("/")[0] == "login" ||
               link.split("/")[0] == "logout" ||
               link.split("/")[0] == "register"
             ){
             delete links[_index]
             return
           }
     ```

   - Khi có một mảng toàn là link chúng ta cần forEach mảng đó ra. Sau đó tìm những phần tử lỗi để lọc nó và xoá nó đi, ở đây mình thấy các value dạng `undefined, "./" , "/", "javascript:void(0)", "login", "logout", "register"` không cần dùng đến nên xoá nó đi.

   - Còn phần lằng ngoằng regex kia là gì?

   - ```javascript
     // Function regex value http https www http://www https://www
     var  _regex = new RegExp("https?:\/\/(?:www\.|(?!www))[a-zA-Z0-9][a-zA-Z0-9-]+[a-zA-Z0-9]\.[^\s]{2,}|www\.[a-zA-Z0-9][a-zA-Z0-9-]+[a-zA-Z0-9]\.[^\s]{2,}|https?:\/\/(?:www\.|(?!www))[a-zA-Z0-9]\.[^\s]{2,}|www\.[a-zA-Z0-9]\.[^\s]{2,}")

     if(!_regex.test(link)){
     links[_index] = url_target + link
     }
     ```

   - Cũng không có gì đâu, vì các link đa phần là tương đối dạng `/phim/xyz` chứ không phải các link `www.phimmoi.net/phim/xyz` nhưng không phải tất cả có một số thì dạng thứ 2 thật, thế nên cần kiểm tra xem nếu link dạng `/phim/xyz` thì chèn thêm `url_target` chính là `"http://www.phimmoi.net/"` vào trước.

   - Cuối cùng là lọc mấy giá trị `null` đi

   - ```javascript
     var links = links.filter(function (link) {
         return link != null;
     });
     ```

   - Vậy là links đã beautiful rồi. Viết nốt hàm để xuất data đó ra file JSON.

   - ```javascript
     // Function export file phimmoi_url.txt
     const exportFile = (data) => {
         fs.writeFile(file_export, data, function(err) {
          if(err) {
              return console.log(err);
          }
          console.log("The file was saved!");
        });
     }
     ```

   - Viết xong rồi giờ chạy thôi.

     ```javascript
     // Main function crawl Page
     getPageContent(url_target).then( (res) => {
       var pageHTML = res.html()
       var links = getPageLink(pageHTML)
       links_filted = filterLink(links)
       return exportFile(JSON.stringify(links_filted))
     })
     ```

   - Chạy hàm `getPageContent` xong thì chạy tiếp hàm `getPageLink` xong rồi thì lấy links ra lọc bằng hàm `filterLink` cuối cùng là xuất ra file JSON `exportFile(JSON.stringify(links_filted))`

   - Nhớ tạo thư mục exports để lưu file nhé, không sẽ lỗi không lưu đc file JSON đấy.

   - ```shell
     mkdir exports
     ```

   - Ra terminal chạy lệnh sau để xem thành quả nhé.

   - ```shell
     node index.js
     ```

3. Kết quả thu được

   ```json
   ["http://www.phimmoi.net/tai-khoan/ho-so.html","http://www.phimmoi.net/tai-khoan/doi-mat-khau.html","http://www.phimmoi.net/tu-phim/","http://www.phimmoi.net/staff/","http://www.phimmoi.net/the-loai/phim-hanh-dong/","http://www.phimmoi.net/the-loai/phim-vien-tuong/","http://www.phimmoi.net/the-loai/phim-chien-tranh/","http://www.phimmoi.net/the-loai/phim-hinh-su/",...]
   ```


### III. Tổng kết

Crawl data ứng dụng được rất nhiều công việc, project này chỉ là demo đơn giản bước đầu để các bạn làm quen, hãy phát triển lên những tool mới chức năng phong phú hơn, đặc biệt hơn và chia sẻ lại cho các bạn khác học hỏi nhé. Đây là bài viết hướng dẫn còn video mình sẽ cập nhật trên [channel Youtube](https://www.youtube.com/channel/UC2Q3C3nxiP9_YWqlR4A6J3g) . Có thắc mắc gì các bạn có thể bình luận ngay bên dưới bài viết này hoặc truy cập [FanPage Facebook](https://www.facebook.com/codethink.io/) để trao đổi thêm.
