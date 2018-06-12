[Source](https://www.codediesel.com/php/unpacking-binary-data/ "Permalink to Unpacking binary data in PHP")

# Giải nén dữ liệu dạng nhị phân trong PHP 

LÀm việc với các file nhị phân trong PHP là yêu cầu hiếm gặp. Tuy nhiên một hàm PHP để 'pack' hoặc 'unpack' khi cần thiết lại có thể giúp bạn rất nhiều. Để cài đặt, chúng ta sẽ bắt đầu giai đoạn làm việc với vấn đề của chương trình, điều này sẽ giúp cho cuộc thảo luận luôn tập trung vào một vấn đề có liên quan. Vấn đề ở đây là: chúng ta muốn viết một hàm lấy một file image như một biến và cho chúng ta biết nó có phải ảnh GIF hay không; không liên quan tới bất kỳ phần mở rộng nào của file. Chúng ta không sử dụng bất kỳ tính năng nào của thư viện GD.

#### A GIF file header Một tiêu đề của file GIF

Với yêu cầu là chúng ta không được sử dụng bất kỳ tính năng đồ hoạ này, để giải quyết vấn đề này chúng ta cần lấy được dữ liệu liên quan từ chính file GIF đó. Không giống như HTML hay XML hay bất kì một định dạng file text nào khác, một file GIF và hầu hết các định dạng file image khác đều được lưu trữ dạng file nhị phân(binary format). Hầu hết các file nhị phân đều có phần header ở đầu file cung cấp các thông tin meta về file cụ thể đó. Chúng ta có thể sử dụng thông tin này để tìm ra kiểu của file hoặc bất cứ thứ gì như là chiều cao, độ rộng của file GIF. Một header của GIF thường có dạng như dưới đây, sử dụng một hex editor như là [WinHex][1]. 

![][2]

Mô tả chi tiết của header được hiển thị như sau:

| ----- |
| 
    
    
    Offset   Length   Contents
      0      3 bytes  "GIF"
      3      3 bytes  "87a" or "89a"
      6      2 bytes  
      8      2 bytes  
     10      1 byte   bit 0:    Global Color Table Flag (GCTF)
                      bit 1..3: Color Resolution
                      bit 4:    Sort Flag to Global Color Table
                      bit 5..7: Size of Global Color Table: 2^(1+n)
     11      1 byte   
     12      1 byte   
     13      ? bytes  
             ? bytes  
             1 bytes   (0x3b)

 | 

Vì vậy để kiểm tra nếu file ảnh là một file GIF hợp lệ, chúng ta cần kiểm tra 3bit đầu tiên của header,được đánh dấu là 'GIF', và với 3 bit tiếp theo, cho ta thông tin về số phiên bản; là '87a' hoặc '89a'. Nó không thể thiếu cho những task giống như là các hàm unpack(). Trước khi ta nhìn vào giải pháp, chúng ta sẽ xem nhanh qua hàm unpack().

#### sử dụng hàm unpack()

[unpack()][3] là sự bổ sung cho [pack()][4] – nó chuyển đổi dữ liệu nhị phân sang một mảng kết hợp dựa trên định dạng có sẵn. Nó là một phần trong các dòng của _sprintf_, chuyển đổi chuỗi data theo một số định dạng nhất định. Đây là 2 function cho phép ta đọc và ghi vào bộ đệm dữ liệu nhị phân theo thành một chuỗi được định dạng trước. Nó dễ dàng cho một lập trình viên trao đổi dữ liệu với chương trình được viết bằng ngôn ngữ khác hoặc định dạng khác. Lấy một vài ví dọ nhỏ.

| ----- |
| 
    
    
    $data = unpack('C*', 'codediesel');
    var_dump($data);

 | 

Nó sẽ in ra như sau, mã thập phân cho 'codediesel' :

| ----- |
| 
    
    
    array
      1 => int 99
      2 => int 111
      3 => int 100
      4 => int 101
      5 => int 100
      6 => int 105
      7 => int 101
      8 => int 115
      9 => int 101
      10 => int 108

 | 

 Trong cả 2 ví dụ trên thì đối số đầu tiên là định dạng string, đối số thứ 2 lại là dữ liệu thực tế. Với mỗi chuỗi định dạng nhất định ứng với cách thể hiện dữ liệu của đối số đó. trong ví dụ này, phần đầu tiên có định dạng là 'C', chỉ định rằng chúng ta xử lý dữ liệu có kí tự đầu tiên như là một byte unsigned. Phần '*' tiếp theo, yêu cầu hàm áp dụng code đã được định dạng trước đó cho tất cả các ký tự còn lại.

Điều này có thể gây nhầm lẫn, phần tiếp theo có thể cung cấp một ví dụ cụ thể hơn.

#### Lấy dữ liệu header

Dưới đây là giải pháp sử dụng hàm unpack() cho các vấn đề về GIF của chúng ta. Hàm _is_gif()_ sẽ trả về true nếu đưa vào file có định dạng GIF.

| ----- |
| 
    
    
    function is_gif($image_file)
    {
     
        /* Open the image file in binary mode */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Read 20 bytes from the top of the file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Create a format specifier */
        $header_format = 'A6version';  # Get the first 6 bytes
    
        /* Unpack the header data */
        $header = unpack ($header_format, $data);
     
        $ver = $header['version'];
     
        return ($ver == 'GIF87a' || $ver == 'GIF89a')? true : false;
     
    }
     
    /* Run our example */
    echo is_gif("aboutus.gif");

 | 

Dòng quan trọng nhất cần lưu ý là định dạng quy định. Các kí tự như 'A6' chỉ định hàm unpack() lấy 6bytes đầu tiên của dữ liệu và thể hiện nó như một chuỗi. Dữ liệu đã truy xuất được lưu trữ vào một mảng liên kết với khoá có tên là 'version'.

Một ví dụ khác được đưa ra dưới đây. Nó trả về một vài dữ liệu bỏ sung trong header của file GIF, bao gồm cả độ rộng và chiều cao của ảnh.

| ----- |
| 
    
    
    function get_gif_header($image_file)
    {
     
        /* Open the image file in binary mode */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Read 20 bytes from the top of the file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Create a format specifier */
        $header_format = 
                'A6Version/' . # Get the first 6 bytes
                'C2Width/' .   # Get the next 2 bytes
                'C2Height/' .  # Get the next 2 bytes
                'C1Flag/' .    # Get the next 1 byte
                '@11/' .       # Jump to the 12th byte
                'C1Aspect';    # Get the next 1 byte
    
        /* Unpack the header data */
        $header = unpack ($header_format, $data);
     
        $ver = $header['Version'];
     
        if($ver == 'GIF87a' || $ver == 'GIF89a') {
            return $header;
        } else {
            return 0;
        }
    }
     
    /* Run our example */
    print_r(get_gif_header("aboutus.gif"));

 | 

Ví dụ trên sau khi chạy sẽ in ra như sau:

| ----- |
| 
    
    
    Array
    (
        [Version] => GIF89a
        [Width1] => 97
        [Width2] => 0
        [Height1] => 33
        [Height2] => 0
        [Flag] => 247
        [Aspect] => 0
    )

 | 

Dưới đâu chúng ta sẽ đi vào chi tiết việc các định dạng thông số làm việc như thế nào. Tôi sẽ chi nhỏ các định dạng, đưa ra các chi tiết cho mỗi định dạng.

| ----- |
| 
    
    
    $header_format = 'A6Version/C2Width/C2Height/C1Flag/@11/C1Aspect';

 | 

| ----- |
| 
    
    
    A - Read a byte and interpret it as a string. 
        Number of bytes to read is given next
    6 - Read a total of 6 bytes, starting from position 0
    Version - Name of key in the associative array where data 
        retrieved by 'A6' is stored
     
    / - Start a new code format
    C - Interpret the next data as an unsigned byte
    2 - Read a total of 2 bytes
    Width - Key in the associative array
     
    / - Start a new code format
    C - Interpret the data as an unsigned byte
    2 - Read a total of 2 bytes
    Height- Key in the associative array
     
    / - Start a new code format
    C - Interpret the data as an unsigned byte
    1 - Read a total of 2 bytes
    Flag - Key in the associative array
     
    / - Start a new code format
    @ - Move to the byte offset specified by the following number.
          Remember that the first position in the binary string is 0. 
    11 - Move to position 11
     
    / - Start a new code format
    C - Interpret the data as an unsigned byte
    1 - Read a total of 1 bytes
    Aspect - Key in the associative array

 | 

Có thể tìm thấy các định dạng tuỳ chọn khác [ở đây][4]. Mặc dù tôi chỉ đưa ra một vài ví dụ nhỏ, hàm pack/unpack có khả năng làm được nhiều hơn những thứ được trình bày ở đây.

Note: Từ phiên bản PHP 7.2.0 kiểu float và double được hỗ trợ bởi cả Big Endian và Little Endian.

[1]: http://www.x-ways.net/winhex/index-m.html
[2]: http://www.codediesel.com/wp-content/uploads/2010/09/winhex.gif "winhex"
[3]: http://php.net/manual/en/function.unpack.php
[4]: http://www.php.net/manual/en/function.pack.php
