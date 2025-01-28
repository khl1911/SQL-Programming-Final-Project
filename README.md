# SQL-Programming-Final-Project
## Giả định bối cảnh
  Với mục đích xem xét và đánh giá khách quan tình hình kinh doanh thực tế của doanh nghiệp và hiệu suất của các nhân sự quản lý khu vực, các lãnh đạo yêu cầu nhân sự phòng Phân tích dữ liệu xây dựng hệ thống báo cáo tổng hợp và phân tích kết quả kinh doanh của toàn doanh nghiệp, từng khu vực hay mỗi nhân sự quản lý chi nhánh.
## Tổng quan về Dữ liệu đầu vào
### fact_kpi_month_raw_data
  Cung cấp chi tiết thông tin dư nợ của từng mã khách hàng trong từng khu vực kinh doanh, bao gồm các cột:
  
    + kpi_month: Năm và tháng ghi nhận thông tin, được trình bày dưới dạng: Năm*100 + Tháng.
    
    + pos_cde: Ký hiệu chi nhánh ghi nhận.
    
    + pos_city: Tỉnh ghi nhận.
    
    + application_id: Mã tài khoản ghi nhận dư nợ.
    
    + outstanding_principal: Dư nợ còn lại của tài khoản.
   
    + write_off_balance_principal: Khoản dư nợ đã ghi nhận Write off (Write off là khoản nợ xấu được doanh nghiệp xác định rằng không thể thu hồi nên không ghi nhận vào dư nợ).
    
    + psdn: Ghi nhận tài khoản có phát sinh dư nợ vào tháng đó.
    
    + max_bucket: Xác định các mức nợ xấu của dư nợ, gồm các nhóm: Null, 1, 2, 3, 4, 5 (Nợ xấu được dùng để tính Non Performing Loan - NPL là những khoản nợ từ nhóm 3 trở lên).
    
### fact_txn_month_raw_data
  
  Ghi nhận các giao dịch phát sinh, bao gồm các cột:
  
    + transaction_date: Ngày phát sinh giao dịch.
    
    + account_code: Mã kế toán dùng để xác nhận giao dịch liên quan đến chỉ tiêu tương ứng trong báo cáo kinh doanh.
    
    + account_description: Mô tả giao dịch phát sinh.
    
    + analysis_code: AAAA.BB.CC.DD.EEE, mã ký hiệu xác định chi nhánh phát sinh giao dịch (AAAA - DVML/HEAD là đơn vị mạng lưới/đơn vị phòng ban, BB - Mã vùng từ 00 đến 06, C - Mã khu vực từ A đến H, DD - Mã tỉnh từ 00 đến 63, EE - Ký hiệu chi nhánh).
    
    + amount: Giá trị giao dịch phát sinh.
    
    + d_c: Gồm hai giá trị: D là khoản chi, C là khoản thu. 

### kpi_asm_data

  Cung cấp hiệu suất các chỉ tiêu của từng Sale Manager, bao gồm các khu vực:
  
    + month_key: Năm và tháng ghi nhận thông tin, được trình bày dưới dạng: Năm*100 + Tháng.
    
    + area_name: Tên khu vực của Sale Manager.
    
    + sale_name: Tên Sale Manager.
    
    + email: Email của Sale Manager.
    
    + month_ltn: 12 cột tương ứng với 12 tháng ghi nhận giá trị Load to new của chi nhánh mà Sale Manager quản lý (Load to new là số dư nợ cho vay khách hàng mới trong tháng).
    
    + month_psdn: 12 cột tương ứng với 12 tháng ghi nhận số khách hàng có phát sinh dư nợ trong tháng đó của chi nhánh mà Sale Manager quản lý. (Mỗi 1 khách hàng mới trong tháng có phát sinh dư nợ sẽ tính là 1)
    
    + month_app_rate: 12 cột tương ứng với 12 ghi nhận tỷ lệ số lượng hồ sơ khách hàng được duyệt/hồ sơ khách hàng đăng ký vay.

## Công cụ sử dụng

  Hệ quản trị cơ sở dữ liệu: PosgreSQL
  
  Công cụ quản lý cơ sở dữ liệu: DBeaver

## Quá trình làm bài
### Tạo bảng chứa thông tin các chỉ tiêu cần báo cáo

  Bảng thông tin cần có tên chỉ tiêu, ID cho các chỉ tiêu tương ứng (Primary key). Từ đó, ta có thể làm việc với các ID - ở dạng số thay vì từng tên chỉ tiêu - ở dạng chữ. Câu lệnh thực hiện như sau:
```sql
  create table dim_indexes(
	  index_id serial primary key,
	  index_name varchar(255) not null,
	  rec_created_dt timestamp default CURRENT_TIMESTAMP,
    rec_updated_dt timestamp default CURRENT_TIMESTAMP
);

 insert into dim_indexes (index_name) values ('1. Lợi nhuận trước thuế'); 
 insert into dim_indexes (index_name) values ('Thu nhập từ hoạt động thẻ'); 
 insert into dim_indexes (index_name) values ('Lãi trong hạn'); 
 insert into dim_indexes (index_name) values ('Lãi quá hạn'); 
 insert into dim_indexes (index_name) values ('Phí Bảo hiểm'); 
 insert into dim_indexes (index_name) values ('Phí tăng hạn mức'); 
 insert into dim_indexes (index_name) values ('Phí thanh toán chậm, thu từ ngoại bảng, khác…'); 
 insert into dim_indexes (index_name) values ('Chi phí thuần KDV'); 
 insert into dim_indexes (index_name) values ('DT Nguồn vốn'); 
 insert into dim_indexes (index_name) values ('CP vốn TT 2'); 
 insert into dim_indexes (index_name) values ('CP vốn TT 1'); 
 insert into dim_indexes (index_name) values ('CP vốn CCTG'); 
 insert into dim_indexes (index_name) values ('Chi phí thuần hoạt động khác'); 
 insert into dim_indexes (index_name) values ('DT Fintech'); 
 insert into dim_indexes (index_name) values ('DT tiểu thương, cá nhân'); 
 insert into dim_indexes (index_name) values ('DT Kinh doanh'); 
 insert into dim_indexes (index_name) values ('CP hoa hồng '); 
 insert into dim_indexes (index_name) values ('CP thuần KD khác '); 
 insert into dim_indexes (index_name) values ('CP hợp tác kd tàu (net)'); 
 insert into dim_indexes (index_name) values ('Tổng thu nhập hoạt động'); 
 insert into dim_indexes (index_name) values ('Tổng chi phí hoạt động'); 
 insert into dim_indexes (index_name) values ('CP thuế, phí'); 
 insert into dim_indexes (index_name) values ('CP nhân viên'); 
 insert into dim_indexes (index_name) values ('CP quản lý'); 
 insert into dim_indexes (index_name) values ('CP tài sản'); 
 insert into dim_indexes (index_name) values ('Chi phí dự phòng'); 
 insert into dim_indexes (index_name) values ('2. Số lượng nhân sự ( Sale Manager )'); 
 insert into dim_indexes (index_name) values ('3. Chỉ số tài chính'); 
 insert into dim_indexes (index_name) values ('CIR (%)'); 
 insert into dim_indexes (index_name) values ('Margin (%)'); 
 insert into dim_indexes (index_name) values ('Hiệu suất trên/vốn (%)'); 
 insert into dim_indexes (index_name) values ('Hiệu suất BQ/ Nhân sự'); 
