# Corrupted hard drive
## Description
File .vhd và file câu hỏi: [here](https://drive.google.com/file/d/1o5400wEi8K26ic3kQpZXg40dgsy8-hWU/view?usp=sharing)
- What is the starting address of the LBA address? Format (0xXXXXX)
- What is the tampered OEM ID? Format (0xXXXXXXXXXXXXXXXX)
- After Fixing the disk, my friend downloaded a file from Google, what is the exact time when he clicked to download that file? Eg: 2024-01-01 01:01:01
- How much time did that file take to for download (in seconds)?? Format  (XXX)
- The first directory he moved this file to?
- Last directory the suspicious move the file to?
- The time he of the deletion?? Eg: 2024-01-01 01:01:01
## Requirements
### Công cụ hỗ trợ
- Công cụ phân tích ổ đĩa: FTK Imager/ Autospy
- Công cụ parsing $MFT, $UsnJrnl:$J: [MFTECmd](https://ericzimmerman.github.io/#!index.md)
- Công cụ parsing $LogFile: [LogFileParser](https://drive.google.com/file/d/1iO7LfmgFlCaP16D0xCBlK2PC0t8_mkcX/view?usp=sharing)
- Công cụ hỗ trợ: HxD, Excel, Notepad
### Objective
- Kiến thức về cấu trúc NTFS
- Khả năng đọc và phân tích file 
## Challenge
### What is the starting address of the LBA address? Format (0xXXXXX)
Địa chị LBA(Logical Block Addressing) là địa chỉ bắt đầu của các Partition. File Deleted.vhd có 1 Partition nên có thể coi như challenge đang yêu cầu địa chỉ LBA của Partition này.

![1](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/1.png)

Có 2 cách để xác định LBA của Partition.

**Cách 1**: Giải quyết nhanh vấn đề nhưng kiến thức không đảm bảo

Vì chỉ có duy nhất 1 Partition nên có thể giả định những byte đầu tiên bắt đầu cho Partition đó là duy nhất. Sử dụng AutoSpy hoặc FTK imager, mình nhấn vào Partition và copy 1 vài byte đầu.

![2](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/2.png)

Sau đó mình quay trở lại với hex view của toàn bộ ổ đĩa, sử dụng chức năng tìm kiếm Binary để tìm ra vị trí của những bytes đó trong ổ đĩa và tham chiếu qua ô địa chỉ để có được LBA address.

![3](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/3.png)

Và như trong hình, mình đã tìm được địa chỉ LBA.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/6.png)

**Cách 2**: Giải quyết được vấn đề áp dụng kiến thức về cấu trúc của MBR, NTFS và tạo tiền đề giải quyết câu 2.

Ở đây có thể sử dụng AutoSpy, FTK imager và HxD đều được. Nhưng mình chọn HxDc chỉnh Offset sang decimal để dễ tham chiếu hơn. Tham khảo Wikipedia, ta có thể phân tích một chút về MBR.

MBR (Master Boot Record) nằm ở sector đầu tiên của ổ đĩa và chiếm 512 byte. Trong MBR, Bootloader sẽ chiếm 446 bytes đầu tiên nên ta sẽ bỏ qua 446 bytes này đề tiến tới Partition table ở 64 byte kế tiếp (trong trường hợp đơn giản, thường sẽ có 4 Partition trong MBR). Dấu hiệu kết thúc của MBR nằm ở 2 byte cuối với giá trị 55 AA.

Bỏ qua 446 bytes đầu. Có thể thấy rõ 2 bytes kết thúc 55 và AA

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/4.png)

16 byte kế tiếp cho ta thông tin về Partition đầu tiên và duy nhất của ổ đĩa.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/5.png)

Byte đầu tiên (00) cho ta biết trạng thái hoạt động của Partition. 80 là active, 00 là inactive. Trong trường hợp này, Partition đang trong trạng thái inactive. 3 byte tiếp theo thể hiện cho địa chỉ CHS của sector đầu tiên trong Partiton nhưng vì không quan trọng nên ta bỏ qua. Byte thứ 5 (07) thể hiện cho hệ thống tệp, 07 đại diện cho hệ thống NTFS, đây có thể là 1 gợi ý để làm các câu sau. 3 byte tiếp theo thể hiện cho địa chỉ CHS của sector cuối cùng  trong Partiton nhưng vì không quan trọng nên ta bỏ qua tiếp. 4 byte kế tiếp (80 00 00 00) cho ta biết địa chỉ LBA của Partition, đây là phần giúp trả lời cho câu 1.

Vì HxD thể hiện byte theo kiêu Little Endian, nên ta cần chỉnh lại để có được kết quả chính xác là 00 00 00 80, tương đương với 128 trong hệ hex. Lấy 128 x 512 = 65536 rồi quy đổi sang hệ hex sẽ cho ra được địa chỉ LBA.

**Kết quả**: 0x10000

### What is the tampered OEM ID? Format (0xXXXXXXXXXXXXXXXX)

Tìm hiểu thêm về vị trí OEM ID trong File System NTFS tại [đây](https://en.wikipedia.org/wiki/NTFS#Structure). Dựa vào thông tin ở câu 1, ta có thể đoán được File System của Partition này là NTFS. Ngoài ra khi tham chiếu tới địa chỉ LBA, Decoded text của 8 byte đầu tiên của Partition sau 3 byte đầu cũng thể hiện 3 chữ NTF. Chữ S bị thiếu thể hiện rằng OEM ID đã bị tampered (sửa đổi). Theo cấu trúc NTFS, 3 byte đầu tiên thể hiện x86 JMP and NOP instructions. 8 byte tiếp theo là OEM ID và đây cũng là kết quả cho câu thứ 2.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/6.png)

Kết quả: 0x4E54460020202020

### After Fixing the disk, my friend downloaded a file from Google, what is the exact time when he clicked to download that file?

Câu này yêu cầu phải suy nghĩ nhiều 1 chút. Câu hỏi cho ta gợi ý về việc sửa lại disk để có thể làm trả lời được câu hỏi. Ta sẽ sửa byte có giá trị offset 00010006 từ 00 thành 53 để trả lại OEM ID chính xác là NTFS như trong phần Partition table đã gợi ý.
- Đối với FTK Imager, ta cần sử dụng 1 công cụ Hex Editor sửa lại OEM ID của disk thì FTK Imager mới có thể nhận diện được File system và đưa ra phân tích.
    - Trước khi sửa:
      ![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/7.png)
    - Sau khi sửa
![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/8.png)

Đối với AutoSpy, ta có thể không cần chỉnh sửa mà sử dụng disk có sẵn để phân tích vì AutoSpy vẫn cho ra được nội dung cân thiết.

Tiếp theo, ta cần tìm ra thời gian của 1 tệp được tải được từ Google. Đầu tiên ta cần xác định tệp được tải xuống từ google là tệp nào trước. Có nhiều cách để làm, mình chọn cách đơn giản nhất là sừ dụng Search trong FTK imager hoặc Keyword search trong AutoSpy để tìm cụm từ “google”. Cả 2 đều trả lại cho minh cùng 1 kết quả là Blue_Team_Notes.pdf. Khi đã xác định được file, việc trả lời câu hỏi sẽ dễ dàng hơn.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/9.png)
![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/10.png)

Việc cần làm tiếp theo là sử dụng 1 công cụ để parse tệp $LogFile. Đây là tệp dùng để ghi lại nhật ký giao dịch của các tệp. Công cụ mình sử dụng là LogFileParser.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/11.png)

Các bước tiếp theo mình sử dụng FTK imager còn AutoSpy có thể được thực hiện tương tự.
Đầu tiên ta cần xuất file $LogFile và $UsnJrnl_$J từ trong ổ đĩa bằng cách nhấn chuột phải vào $LogFile và chọn Export Files và để ở nơi dễ thấy.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/12.png)

Sau đó ta chọn $LogFile đã được xuất vào LogFileParser và chờ kết quả. Có thể thấy $LogFile sau khi được parse trả về rất nhiều tệp csv. Tuy nhiên mình chỉ chú ý 1 tệp quan trọng là LogFile_lfUsnJrnl.csv vì đây là tệp $UsnJrnl_$J mà mình sẽ nói rõ hơn ở dưới. 

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/13.png)

VÌ các tệp được xuất là tệp csv nên mình sử dụng Excel để dễ nhìn hơn. Ngoài ra, chúng ta còn có thể sử dụng công cụ MFTECmd để parser file $UsnJrnl_$J, đây cũng chính là tệp mà mình đã chọn khi parse bằng LogFileParser ở trên và mình sẽ sử dụng tệp này cho các phân tích tiếp theo. $UsnJrnl_$J chứa thông tin về tất cả các tệp đã được thay đổi trong hệ thống tệp và lý do thay đổi. 

Cú pháp sử dụng MFTECmd để parser file $UsnJrnl_$J:
```
	MFTECmd.exe -f $UsnJrnl_$J --csv .
```
![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/14.png)
![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/15.png)

Khi 1 file được tải về từ Google, 1 file tạm sẽ được tạo cho đến khi quá trình tải hoàn tất, Có 2 lần đổi tên từ khi tạo file cho đến khi file được tải xong. Để kết quả chính xác nhất, mình sẽ chọn Timestamp ngay cả khi file mới được tạo và tên chỉ là 1 chuỗi kí tự.

**Kết quả**: 2024-10-22 21:51:13

### How much time did that file take to for download (in seconds)?? Format  (XXX)
![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/16.png)

Thời gian tải file là từ 10/22/2024 21:51.13 đến 10/22/2024 21:53.19, tương đương 126s.
Đây là thời gian kể từ khi file Blue_Team_Notes.pdf.crdownload đi từ DataExtend cho đến DataExtend|Close

**Kết quả**: 126

### The first directory he moved this file to?

Ta có thể sử dụng thêm tệp $MFT để hỗ trợ quá trình. Tệp $MFT chứa Master File Table, là nơi lưu trữ thông tin về các tệp.

Mỗi tệp hoặc thư mục trong File System NTFS đều có một entry riêng trong MFT, và EntryNumber là đặc trưng cho mỗi entry. EntryNumber có thẻ được sử dụng lại, tuy nhiên EntryNumber phải là duy nhất tại một thời điểm nhất định. Nếu EntryNumber thể hiện cho 1 tệp thì ParentEntryNumber sẽ thể hiện cho thư mục chứa tệp đó. SequenceNumber cho ta biết số lần mà EntryNumber đã được sử dụng lại. Sự kết hợp giữa EntryNumber và SequenceNumber thường sẽ đại diện cho 1 đối tượng duy nhất ngay cả khi đối tượng bị xóa. 

Sau khi lướt thêm file $UsnJrnl:$J, mình bắt gặp 1 loạt entry liên quan tới file Blue_Team_Notes.pdf, ở đây mình thấy được sự thay đổi về vị trí thông qua UpdateReasons và ParentEntryNumber. 

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/16.png)

Lúc đầu file Blue_Team_Notes.pdf được tải về có ParentEntryNumber| SequenceNumber là 45| 1 tương đương với thư mục đầu tiên mà tệp được tải xuống là thư mục docs.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/18.png)

Sau đó ParentEntryNumber| SequenceNumber được đổi sang 56| 2 tương đương với việc tệp đã được di chuyển tới thư mục khác. Thêm vào đó cột UpdateReasons của 3 hàng có giá trị lần lượt là RenameOldName, RenameNewName, và RenameNewName|Close, đây là những dấu hiệu cho thấy tệp đã được đổi tên. Nên nhớ việc thay đổi vị trí của 1 đối tượng trong ổ đĩa đồng nghĩa với việc tên tệp cũng được thay đôi. 

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/19.png)

Sau khi kiểm EntryNumber| SequenceNumber với giá trị là 56| 2, ta có thể thấy được thư mục đầu tiên mà tệp được chuyển tới sau khi tải xuống như sau.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/20.png)

**Kết quả**: best

### Last directory the suspicious move the file to?

Câu này giống có cách làm giống với câu 5. Ta có thể quan sát thấy tệp Blue_Team_Notes.pdf được đổi thư mục một vài lần nữa trước khi bị xóa và chuyển vào Recycle Bin. Nhiệm vụ của chúng ta là tìm được thư mục cuối cùng mà tệp được chuyển vào trước khi bị xóa. Theo như hình ta có thể thấy giá trị ParentEntryNumber| SequenceNumber cuối cùng được thể hiện là 47|1.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/22.png)

Sau đo mình tra cặp EntryNumber| SequenceNumber với giá trị 47| 1 và nhận được kết quả như sau.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/21.png)

**Kết quả**: MustRead

### The time he of the deletion?? Eg: 2024-01-01 01:01:01

Sau khi đi qua hết tất cả câu hỏi ở trên thì câu này khá đơn giản để trả lời. Chỉ cần lấy TimeStamp của hàng có UpdateReasons là FileDelete| Close là xong.

![](https://github.com/tlmt009147/isitdtu2024-corrupted-hard-drive/blob/ae36c7a242189f1c30b51e19ab0270d4010e7843/writeup/image/22.png)

**Kết quả**: 2024-10-22 22:20:28

## P/S

Cảm ơn anh Triet Tran đã hỗ trợ mình rất nhiều khi làm bài này.







