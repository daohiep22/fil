chạy file main.py

có kèm sơ đồ class và sơ đồ hoạt động

formdata.py chứa một số chuẩn dữ liệu

ý tưởng:
Simulator thực hiện các thao tác chính
Application là interface, các realization của nó sẽ đại diện cho các dạng SFC có topo khác nhau (đường thẳng như SimpleApplication, waxman (chưa làm))
Distribution: cho biết khoảng thời gian giữa 2 SFC
Selector: thuật toán đặt VNF lên server, SimpleSelector.analyse() sử dụng random, chỉ để test input output, không cần đọc
Logger: thực hiện việc log
các mối quan hệ multiplicity được thể hiện trong diagram.jpg

requestRoom và readyRoom là tài nguyên dạng Store của Simpy, lưu và lấy dữ liệu ra theo kiểu First In First Out, các phương thức .put(object) để đẩy 1 object vào và .get() để lấy ra



Luồng hoạt động:
sim.run(), thực hiện gắn 3 tiến trình là generator, handler, deployer vào env (theo Simpy), 3 hàm này sẽ chạy rời rạc coi như độc lập với nhau, theo ý tưởng của Simpy là thế

generator(app): hard cứng 100% không sửa thêm
tạo ra 1 SFC theo format của app quy định sau mỗi khoảng thời gian tuân theo phân phối app.distribution, sfc tạo ra được đẩy vào requestRoom, nằm trong hàng đợi chờ handler() xử lí, hàng đợi nhưng thực ra không có độ trễ nào ở đây
waxman topo của SFC sẽ được tạo trong hàm Application.generate_SFC()

handler(app): hard cứng 100% không sửa thêm
lấy sfc vừa tạo ra từ requestRoom, thực hiện hàm app.selector.analyse(topo, sfc), dựa vào topo vật lí và sfc["structure"] để xem xét xem sfc này có deploy được lên hệ thống không, nếu có thì app.selector.analyse(topo, sfc) trả về deploy là thông tin để deploy và đẩy thông tin này vào readyRoom, không thì chả làm gì, mặc định là bị drop

deployer(): hard cứng 90%, tuỳ vào node có bao nhiêu thông số thì thêm vài dòng vào
lấy thông tin deploy từ readyRoom, và cập nhật thông số cho các node của topo vật lí

remover(): hard cứng như deployer
mỗi sfc sau một khoảng thời gian TTL sẽ bị xoá và giải phóng tài nguyên cho các node

từ file results/result_event.csv có thể suy ra được công suất, tuy nhiên trong code đã có hàm để tính công suất trong lúc code chạy và log kết quả ra results/result_energy.csv



problem:
1, phân phối poisson lamda = 8 trong 60 phút, random các giá trị trong khoảng thực [0, +vc), sau đó làm tròn giá trị này, một số trường hợp sẽ trả về giá trị là 0 tức là 2 sfc đến cùng 1 thời điểm, và nếu có 3 hay 4 sfc đến cùng 1 thời điểm thì code sẽ chạy sai.
-> khắc phục
+) 1: xem xét sửa code, cụ thể là gộp deployer và handler
+) 2: Poisson.next() cộng thêm 1 vào giá trị return là không bao giờ = 0 như đã thực hiện, hoặc làm các biện pháp tương tự, tuy nhiên có ảnh hưởng về mặt toán học hay không thì chưa rõ

