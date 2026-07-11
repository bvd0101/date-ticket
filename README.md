HƯỚNG DẪN CÀI ĐẶT HỆ THỐNG VÉ HẸN HÒ
Hệ thống này sử dụng giao diện tĩnh kết hợp với Google Apps Script (gửi mail) và Cloudflare Workers (trung chuyển API). Làm đúng thứ tự dưới đây để tự tạo hệ thống nhận vé cho riêng bạn.


Bước 1: Thiết lập nơi nhận Email (Google Apps Script)
Truy cập script.google.com và tạo một Dự án mới.
Xóa mã mặc định, dán đoạn mã này vào:

```
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var myEmail = "ĐIỀN_EMAIL_CỦA_BẠN_VÀO_ĐÂY@gmail.com"; 
    var guestEmail = data.guestEmail || "";
    var toEmail = guestEmail !== "" ? myEmail + "," + guestEmail : myEmail;

    var subject = "DATE TICKET !!";
    var body = "Ngày: " + data.date + "\nGiờ: " + data.time + "\nKế hoạch: " + data.food;
    
    var imageBlob = null;
    if (data.image) {
      var decoded = Utilities.base64Decode(data.image.split(",")[1]);
      imageBlob = Utilities.newBlob(decoded, 'image/png', 'DateTicket.png');
    }

    if (imageBlob) {
      MailApp.sendEmail({ to: toEmail, subject: subject, body: body, attachments: [imageBlob] });
    } else {
      MailApp.sendEmail(toEmail, subject, body);
    }
    return ContentService.createTextOutput(JSON.stringify({"status": "ok"})).setMimeType(ContentService.MimeType.JSON);
  } catch(error) {
    return ContentService.createTextOutput(JSON.stringify({"error": error.toString()})).setMimeType(ContentService.MimeType.JSON);
  }
}
```

Thay dòng ĐIỀN_EMAIL_CỦA_BẠN_VÀO_ĐÂY... thành email thực tế của bạn. Bấm Ctrl + S để lưu.
Bấm Triển khai (Deploy) -> Bản triển khai mới (New deployment).
Chọn loại Ứng dụng web (Web app). Quyền truy cập bắt buộc chọn: Bất kỳ ai (Anyone).
Cấp quyền truy cập Gmail theo yêu cầu. Copy URL Ứng dụng web lưu ra nháp.


Bước 2: Thiết lập cổng trung chuyển (Cloudflare Workers)
Đăng nhập dash.cloudflare.com, vào mục Workers & Pages -> Tạo Worker mới.
Bấm Edit code, xóa mã mặc định và dán đoạn này vào:

```
export default {
  async fetch(request, env) {
    const allowedOrigin = "https://TEN_TAI_KHOAN_GITHUB.github.io"; // ĐIỀN LINK GITHUB PAGES CỦA BẠN VÀO ĐÂY
    const origin = request.headers.get("Origin");

    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: { "Access-Control-Allow-Origin": allowedOrigin, "Access-Control-Allow-Methods": "POST, OPTIONS", "Access-Control-Allow-Headers": "Content-Type" }
      });
    }
    if (origin !== allowedOrigin) return new Response("Cấm truy cập", { status: 403 });

    if (request.method === "POST") {
      const gasUrl = "DÁN_URL_GOOGLE_SCRIPT_CỦA_BẠN_VÀO_ĐÂY"; 
      const gasResponse = await fetch(gasUrl, { method: 'POST', body: await request.text() });
      const finalResponse = new Response(gasResponse.body, gasResponse);
      finalResponse.headers.set("Access-Control-Allow-Origin", allowedOrigin);
      return finalResponse;
    }
    return new Response("Not found", { status: 404 });
  }
};
```

Sửa đúng 2 dòng tôi đã ghi chú viết hoa trong mã. Tuyệt đối không để dấu / ở cuối các đường link.
Bấm Deploy. Copy đường link Worker (có đuôi .workers.dev).


Bước 3: Gắn API vào mã nguồn Frontend
Mở tệp index.html tải từ kho lưu trữ này.
Cuộn xuống dòng số 384, tìm hàm fetch("...").
Thay thế đường link ảo bằng cái đường link Worker bạn vừa copy ở Bước 2.
Lưu tệp lại.


Bước 4: Triển khai và lấy Link gửi đi
Tạo một kho lưu trữ Public mới trên tài khoản GitHub của bạn.
Upload tệp index.html đã sửa lên đó.
Vào tab Settings -> Pages, mục Branch chọn nhánh main và bấm Save.
Chờ 1 phút để GitHub xuất bản trang web.
Mở link đó bằng tab ẩn danh, điền thử thông tin và kiểm tra hộp thư. Mail nổ thì copy cái link trang web đó gửi cho người bạn muốn rủ đi chơi.


### Ủng hộ tác giả (Donate)
Nếu bạn thấy mã nguồn này hữu ích và muốn mời tôi một ly cà phê, có thể chuyển khoản qua thông tin dưới đây:

*   **Ngân hàng:** [Techcombank]
*   **Số tài khoản:** [9868633333]
*   **Chủ tài khoản:** [BUI VAN DIEP]


