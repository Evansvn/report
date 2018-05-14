# report
synthesis all of things that I did in a week.
# SDNet
##	SDNet là gì?
-	Xilinx SDNet data plane builder tạo ra các hệ thống có thể được lập trình cho một loạt chức năng xử lý gói (phân loại, chỉnh sửa gói từ đơn giản đến phức tạp). 
-	Có thể sử dụng ngôn ngữ PX hoặc P4 để thực hiện các chức năng xử lý gói.
##	Chức năng SDNet?
-	Xây dựng 1 hệ thống SDNet phân cấp, gồm nhiều engine khác nhau như:
+ Parsing engines: tách thông tin Header hoặc data từ packets.
+ Editing engines: thao tác với nội dung của packets như chèn, sửa đổi, xóa dữ liệu trong packets.
+ Lookup engines: khởi tạo các IP core được tạo ra từ thư viện các basic type cho quá trình xử lý gói như: EM (exact match), LPM (Longest-prefix Match), TCAM….
+  Tuple engines: thao tác các tuple/metadata (dữ liệu/siêu dữ liệu) được xác định từ packets, từ bên ngoài hoặc từ các công cụ khác.
+ User engines: cung cấp một cơ chế để kết hợp IP tùy chỉnh của người dung, phù hợp với giao diện engines SDNet.
+ Systems: có thể được phân cấp để thực hiện các chức năng xử lý gói lớn và phức tạp.
-	Hỗ trợ cho 3 miền clock khác nhau để các engine có thể  run tại các tần số khác nhau như:
+ “Line rate” (tốc độ dòng) của bus dữ liệu gói được sử dụng cho các engine đọc và sửa đổi gói.
+ “Packet rate” ( tốc độ gói) được dung cho các chức năng xảy ra một lần như individual lookup. 
+ “Control rate” là tốc độ của giao diện điều khiển của bộ nhớ được ánh xạ để điều khiển và cấu hình engines.
-	Các hệ thống dẫn đến việc triển khai phần cứng hiệu suất cao, đạt được khoảng 10-100 Gb / s.
-	Khả năng backpressure hệ thống được tự động tạo ra bao gồm cả việc chèn bộ đệm cung cấp đồng bộ hóa dữ liệu cho engines. Khả năng này cho phép backpressure bus gói cho xử lý gói tạm thời.
-	Một mô hình C ++ được tạo ra để mô phỏng hệ thống mức cao của đặc tả trước khi chạy mô phỏng RTL.
-	SDNet hỗ trợ giao thức truyền tín hiệu LBUS hoặc AXI-Stream cho các giao diện gói
##	Yêu cầu xử lý gói và funtions.
Các yêu cầu xử lý gói được cung cấp bởi người dùng và phải bao gồm:
• Các định dạng gói được phân tích cú pháp và / hoặc chỉnh sửa
• Các trường bit hoặc trường con được trích xuất (trình phân tích cú pháp), hoặc được chèn, xóa hoặc sửa đổi (trình soạn thảo)
• Định dạng khóa đầu ra (kết quả phân tích / phân loại)
•	SDNet Dataflow Model
1.	Port:
+ Packet Ports: là giao diện SDNet chính chịu trách nhiệm di chuyển xung quanh các packets giữa các engine cũng như các khối bên ngoài. Hiện tại, các engine chỉ chứa tối đa 1 input pakets và 1 output packet. Parsing engines, editing engines và systems yêu cầu packet port. User engines có thể lựa chọn packet ports. Còn các engine khác không có packet port.
+ Tuple Ports: là giao diện SDNet phụ chịu trách nhiệm cho việc truyền metadata liên quan đến packet giữa các engines và bên ngoài. Các engine có thể chứa nhiều Tuple ports khác nhau.
+ Access ports: là các cổng điều khiển ánh xạ bộ nhớ, được kết nối tự động bởi trình biên dịch.
+ Plain Ports: được sử dụng trong giao tiếp RTL giữa các user engine, không bị bắt buộc bởi 3 loại port trên. 
2.	Engines:
+ Parsing Engine
+ Editing engine
+ Tuple engine
+ Lookup Engine: EM, LPM, TCAM, DIRECT
+ User Engine
+ System

 
# PX
## Giới thiệu PX
+ Định nghĩa PX:
- PX là ngôn ngữ lập trình bậc cao cho thành phần Programable Packet Process (PP) của Xilinx SDNet. 
- PX cho phép người dung tập trung vào các chức năng xử lí gói tin mà không cần quan tâm sâu đến cách thực hiện để đạt hiệu suất cao. PX là ngôn ngữ khai báo, nó quan tâm đến xử lí gói gì hơn là xử lí gói đó như thế nào.
- Một chương trình PX bao gồm các luật để xử lí gói tin, nó gồm 2 đối tương cơ bản là: Engine và Interface. Engine có chức năng để xử lí gói tin còn Interface cho phép giao tiếp giữa các engines với thế giới bên ngoài.
+ Mục đích:
-	Sinh ra code RTL có thể tổng hợp cho kiến trúc của một PPP
-	Thay đổi firmware cho một PPP đã tồn tại. 
## Khai báo kiểu cấu trúc (struct)
-	Định nghĩa dạng của 1 cấu trúc dữ liệu để được phân tích và gồm 1 hoặc nhiều chuỗi dữ liệu.
VD: 1 header MAC Ethernet gồm:
• destination MAC address (48 bits)
• source MAC address (48 bits)
• type field (16 bits)
	Được định nghĩa như sau:
struct {
 dmac : 48,
 smac : 48,
 type : 16
}
## Khai báo class:
Format:
class classIdentifier :: parentClassIdentifier parameterList {
Statements
}
Trong đó:
-	classIdentifier: là tên của class được trình bày. 
-	parentClassIdentifier: Là tên của class cha mà xây dựng nên class đang xét.
-	 parameterList: Là một danh sách các tham số cụ thể cho class cha (Param1, … , ParamN)
-	Statements: gồm các khối khai báo và định nghĩa phương thức, được sắp xếp túy ý cho phù hợp chức năng. Định nghĩa phương thức có định dạng chung sau:
method methodIdentifier = methodBody ;
	MethodIdentifier luôn là một định danh được tích hợp liên kết với lớp cha
	Trong một vài trường hợp đặc biệt đối với một số phương thức đã xác định, phần = methodBody là tùy chọn.

## Khai báo Subclass Engine:
Có 5 lớp engine cha được tích hợp sẵn: ParsingEngine, TupleEngine, EditingEngine, LookupEngine và UserEngine.
### Khai báo ParsingEngine Subclass:
Lớp này có tham số parameterList như sau:
	(maxPacketRegion, maxSectionDepth, firstSection)
-	maxParseRegion: xác định phần gói tối đa ban đầu được phân tích bởi Engine.
-	maxSectionDepth: Xác định số lượng tối đa các packet section (protocol levels) engine truyền tải.
-	firstSection: là tên của 1 phân lớp được khai báo trong mô tả. Đây là section đầu tiên được truyền qua.

Phần body chứa các khối khai báo và không có định ngĩa phương thức.
### Khai báo TupleEngine Subclass:
Lớp này có tham số parameterList như sau:
	(maxSectionDepth, firstSection)
-	maxSectionDepth: một giá trị số nguyên dương không đổi xác định số lượng tối đa các phần xử lý tuple được truyền tải bởi engine.
-	firstSection: là tên của 1 phân lớp được khai báo trong khai báo này. Đây là section đầu tiên được duyệt qua.
Phần body chứa các khối khai báo và không có định ngĩa phương thức.
### Khai báo EditingEngine Subclass:
Lớp này có tham số parameterList như sau:
	(maxPacketRegion, maxSectionDepth, firstSection)
-	maxParseRegion: xác định phần gói tối đa ban đầu được phân tích bởi Engine.
-	maxSectionDepth: Xác định số lượng tối đa các packet section (protocol levels) engine truyền tải.
-	firstSection: là tên của 1 phân lớp được khai báo trong mô tả. Đây là section đầu tiên được truyền qua.
Phần body chứa các khối khai báo và không có định ngĩa phương thức.
### Khai báo LookupEngine Subclass:
Lớp này có tham số parameterList như sau:
(lookupType,capacity,keyWidth,valueWidth,responseType,external)
Trong đó:
-	lookupType: kiểu lookup (EM, LPM, TCAM, DIRECT)
-	capacity: giá trị nguyên dương không đổi xác định số lượng mục nhập tối đa được lưu trữ trong công cụ tra cứu.
-	keyWidth: giá trị nguyên dương không đổi xác định chiều rộng, tính bằng bit, của khóa tìm kiếm để tra cứu
-	valueWidth: giá trị nguyên dương không đổi xác định chiều rộng, tính bằng bit, của kết quả tìm kiếm từ một tra cứu
-	responseType: giá trị số nguyên không âm, trong khoảng 0 đến 2, xác định định dạng của phản hồi được trả về bởi cá thể của công cụ tra cứu. 0 cho biết giá trị chỉ, 1 cho biết một bit hit / miss theo sau là giá trị, và 2 chỉ ra một bit hit / miss, sau đó là giá trị theo sau là khóa. Nếu tham số này vắng mặt, mặc định là một bit hit / miss theo sau bởi giá trị.
-	external: là một giá trị số nguyên không đổi âm, hoặc là 0 hoặc 1, xác định xem thực hiện cá thể của công cụ tra cứu có nằm ngoài trường hợp hệ thống PPP tổng thể hay không. 0 cho biết không phải bên ngoài và 1 chỉ ra bên ngoài. Nếu tham số này vắng mặt, mặc định là không nằm ngoài.
Phần thân của khai báo lớp con LookupEngine chứa một khối khai báo và hai định nghĩa phương thức. 
-	Hai định nghĩa phương thức như sau:
method send_request = {
 key = identifier
}
Và
method receive_response = {
 identifier = value
}
### Khai báo UserEngine Subclass
Lớp này có tham số parameterList như sau:
(maxLatency,controlWidth) Trong đó:
-	maxLatency: phải có mặt và là giá trị nguyên dương không đổi, xác định độ trễ, được đo theo chu kỳ tương ứng với đồng hồ được chỉ định tại thời gian biên dịch, giữa sự xuất hiện của đầu vào mới nhất và tính sẵn có của đầu ra mới nhất
-	controlWidth: là một giá trị số nguyên không âm, chỉ định chiều rộng của giao diện điều khiển của động cơ theo bit, với 0 cho biết không có giao diện điều khiển. Nếu tham số này vắng mặt, mặc định không có giao diện điều khiển.
Phần thân của khai báo lớp con LookupEngine chứa một khối khai báo và không có định nghĩa phương thức.
## Khai báo Interface Class:
### Khai báo Packet Subclass:
Lớp này có tham số parameterList như sau:
(direction)
Với direction có thể là in (chỉ input), out (chỉ output) hoặc inout (cả input lẫn output).
### Khai báo Tuple Subclass:
Lớp này có tham số parameterList như sau:
(direction)
Với direction có thể là in (chỉ input), out (chỉ output) hoặc inout (cả input lẫn output).

Phần thân của khai báo lớp con Tuple chứa một khối khai báo và không có định nghĩa phương thức.
### Khai báo Plain Subclass:
Lớp này có tham số parameterList như sau:
(direction,width)
Với:
-	direction có thể là in (chỉ input), out (chỉ output) 
-	là một giá trị nguyên dương không đổi xác định chiều rộng của giao diện theo bit.
Phần thân của khai báo lớp con Plain là rỗng

## Khai báo Section Subclass:
Lớp này có tham số parameterList như sau:
(level)
Với level chỉ rõ các mức truyền tải của phần mà loại phần này có thể xảy ra trong bất kỳ chuỗi các phần của gói dữ liệu đầu vào.
Phần thân của khai báo phân lớp này có chứa một khối khai báo và ít nhất một phương thức định nghĩa.
### Khai báo Map:
Khối khai báo có thể chứa 1 hoặc nhiều khối khai báo Map có cấu trúc như sau:
map mapIdentifier {
 (key1, result1),
 (key2, result2),
 ...
 (keyn, resultn),
 defaultresult
}
Trong đó: 
-	Key: một hằng số nguyên không âm và tất cả các khóa phải là duy nhất
-	Result: là tất cả các số phân định của phân lớp hoặc tất cả các hằng số nguyên không âm, tùy thuộc vào hoàn cảnh cụ thể trong map được sử dụng.
### Update Method:
method update = {
 destfield1 = valueexpr1,
 ... ,
 destfieldn = valueexprn
}
Được sử dụng để cập nhật hoặc gán các tuple.
### Move_to_Section Method:
method move_to_section = classexpr ;
	Chỉ định Section tiếp theo để thực hiện (có thể giống Section hiện tại).
### Increment_offset, Set_offset, And Set_virtual_offset, Methods
method increment_offset = valueexpr ;
Phương thức increment_offset chỉ định số bit cần bỏ qua trong gói để di chuyển từ phần hiện tại đang được xử lý sang phần tiếp theo sẽ được xử lý, sử dụng một biểu thức giá trị.
method set_offset = valueexpr ;
Xác định sẵn lượng bits sẽ bỏ qua trong gói đầu vào để di chuyển từ Section hiện tại đang được xử lí sang Section tiếp theo sẽ được xử lí.
	
### Methods thêm vào và xóa đi cho Editing Engines
method remove = valueexpr ;
Phương thức remove xác định rằng một số bit dữ liệu sẽ được loại bỏ tại offset hiện tại trong gói, với một biểu thức giá trị cho biết số bit. Biểu thức giá trị đặc biệt rop () có thể được sử dụng để loại bỏ tất cả các bit còn lại của gói.

method remove ;

method insert = valueexpr ;
Phương thức chèn quy định rằng một số bit dữ liệu sẽ được chèn vào trước giá trị offset hiện tại trong gói, với một biểu thức giá trị biểu thị số bit. Các bit được chèn được xác định như sau (trong đó kích thước struct được định nghĩa là zero nếu không có cấu trúc nào có trong Section)

method insert ;

### Khai báo System Subclass 
method connect = {
 destInterface1 = sourceInterface1,
 ... ,
 destInterfacen = sourceInterfacen
}
### Khai báo Constant
const identifier = value;
### Khai báo Object:
classIdentifier objectIdentifier1, …, objectIdentifiern; 

