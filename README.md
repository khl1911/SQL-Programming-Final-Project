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

  Công cụ Visual: Power BI

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
-- Nhập dữ liệu vào bảng
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
```
### Mã hóa các mã giao dịch thành từng chỉ tiêu tương ứng

Từ các ID đã gán cho các chỉ tiêu ở trên, ta tiến hành mã hóa các mã kế toán (cột **account_code** trong bảng **fact_txn_month_raw_data**) với các chỉ tiêu tương ứng để tránh truy vấn trên cột **account_code** với số lượng mã kế toán rất lớn. Chi tiết thực hiện như sau:

```sql
alter table fact_txn_month_raw_data add index_id int4 null;
update fact_txn_month_raw_data
			set
				index_id = CASE
			        -- Lãi trong hạn
			        WHEN account_code IN (702000030002, 702000030001, 702000030102) THEN 3
			
			        -- Lãi quá hạn
			        WHEN account_code IN (702000030012, 702000030112) THEN 4
			
			        -- Phí bảo hiểm
			        WHEN account_code = 716000000001 THEN 5
			
			        -- Phí tăng hạn mức
			        WHEN account_code = 719000030002 THEN 6
					
			        -- Phí thanh toán chậm, thu từ ngoại bảng
			        WHEN account_code IN (719000030003, 719000030103, 790000030003, 790000030103, 790000030004, 790000030104) THEN 7
					--DT Nguồn vốn
			        when account_code in (
			        	702000040001,702000040002,703000000001,703000000002,703000000003,703000000004, 
			        	721000000041,721000000037,721000000039,721000000013,721000000014,721000000036,723000000014, 723000000037,
			        	821000000014,821000000037,821000000039,821000000041,821000000013,821000000036,
				        823000000014,823000000037,741031000001,741031000002,841000000001,841000000005,841000000004,
				        701000000001,701000000002,701037000001,701037000002,701000000101
				        ) then 9 
			        -- CP vốn CCTG
			        WHEN account_code IN (803000000001) THEN 12
			
			        -- CP vốn TT1
			        WHEN account_code IN (802000000002, 802000000003, 802014000001, 802037000001) THEN 11
			
			        -- CP vốn TT2
			        WHEN account_code IN (801000000001, 802000000001) THEN 10
			
			        -- CP hoa hồng
			        WHEN account_code IN (816000000001, 816000000002, 816000000003) THEN 17
			
			        -- CP thuần KD khác
			        WHEN account_code IN (
			            809000000002, 809000000001, 811000000001, 811000000102, 811000000002,
			            811014000001, 811037000001, 811039000001, 811041000001, 815000000001,
			            819000000002, 819000000003, 819000000001, 790000000003, 790000050101,
			            790000000101, 790037000001, 849000000001, 899000000003, 899000000002,
			            811000000101, 819000060001
			        ) THEN 18
			
			        -- DT kinh doanh
			        WHEN account_code IN (
			            702000010001, 702000010002, 704000000001, 705000000001, 709000000001,
			            714000000002, 714000000003, 714037000001, 714000000004, 714014000001,
			            715000000001, 715037000001, 719000000001, 709000000101, 719000000101
			        ) THEN 16
			
			        -- CP thuế, phí
			        WHEN account_code IN (831000000001, 831000000002, 832000000101, 832000000001, 831000000102) THEN 22
			
			        -- CP nhân viên
			        WHEN account_code::text LIKE '85%' THEN 23
			
			        -- CP quản lý
			        WHEN account_code::text LIKE '86%' THEN 24
			
			        -- CP tài sản
			        WHEN account_code::text LIKE '87%' THEN 25
			
			        -- Chi phí dự phòng
			        WHEN account_code IN (
			            790000050001, 882200050001, 790000030001, 882200030001, 790000000001,
			            790000020101, 882200000001, 882200050101, 882200020101, 882200060001,
			            790000050101, 882200030101
			        ) THEN 26
			
			        -- Nếu không thuộc bất kỳ trường hợp nào, gán NULL
			        ELSE NULL
    			end;
```

### Mã hóa mã khu vực

Ở bảng **fact_txn_month_raw_data**, cột **analysis_code** xác định giao dịch phát sinh ở tỉnh nào thông qua mã số **DD** của giá trị AAAA.BB.CC.DD.EEE. Từ đó, mã hóa các tỉnh thành các khu vực tương ứng với các giá trị từ **A** đến **H**. Chi tiết như sau:
```sql
alter table fact_txn_month_raw_data add area_code varchar(50) null;
update fact_txn_month_raw_data 
	set area_code = CASE
	-- Lấy phần tử thứ 4 sau "."
				        -- Khu vực I - Đông Bắc Bộ
				        WHEN split_part(analysis_code, '.', 4) IN ('21', '59', '43', '54', '03', '13', '36', '02', '48') THEN 'B'
				        -- Khu vực II - Tây Bắc Bộ
				        WHEN split_part(analysis_code, '.', 4) IN ('37', '63', '17', '51', '29', '34') THEN 'C'
				        -- Khu vực III - Đồng Bằng Sông Hồng
				        WHEN split_part(analysis_code, '.', 4) IN ('23', '26', '61', '05', '30', '25', '53', '39', '41', '22') THEN 'D'
				        -- Khu vực IV - Bắc Trung Bộ
				        WHEN split_part(analysis_code, '.', 4) IN ('55', '40', '24', '45', '49', '56') THEN 'E'
				        -- Khu vực V - Nam Trung Bộ
				        WHEN split_part(analysis_code, '.', 4) IN ('14', '46', '47', '07', '44', '31', '42', '10', '33', '20', '15', '16', '35') THEN 'F'
				        -- Khu vực VI - Miền Tây Nam Bộ
				        WHEN split_part(analysis_code, '.', 4) IN ('12', '38', '19', '57', '01', '06', '60', '58', '27', '32', '50', '04', '11') THEN 'G'
				        -- Khu vực VII - Miền Đông Nam Bộ
				        WHEN split_part(analysis_code, '.', 4) IN ('28', '62', '08', '09', '18', '52') THEN 'H'
				        -- Đơn vị phòng ban - HEAD
				        ELSE 'A' 
	   				end;
```

Với bảng **fact_kpi_month_raw_data**, tỉnh được xác định thông qua cột **pos_city** do đó chỉ cần mã hóa thẳng trên các giá trị của cột này. Chi tiết như sau:
```sql
alter table fact_kpi_month_raw_data add area_code varchar(50) null;
update fact_kpi_month_raw_data 
	set area_code = CASE 
			        -- Khu vực I - Đông Bắc Bộ
			        WHEN pos_city IN ('Hà Giang', 'Tuyên Quang', 'Phú Thọ', 'Thái Nguyên', 
			                           'Bắc Kạn', 'Cao Bằng', 'Lạng Sơn', 'Bắc Giang', 
			                           'Quảng Ninh') THEN 'B'
			        -- Khu vực II - Tây Bắc Bộ
			        WHEN pos_city IN ('Lào Cai', 'Yên Bái', 'Điện Biên', 'Sơn La', 'Hòa Bình','Lai Châu') THEN 'C'
			        -- Khu vực III - Đồng Bằng Sông Hồng
			        WHEN pos_city IN ('Hà Nội', 'Hải Phòng', 'Vĩnh Phúc', 'Bắc Ninh', 'Hưng Yên', 
			                           'Hải Dương', 'Thái Bình', 'Nam Định', 'Ninh Bình', 
			                           'Hà Nam') THEN 'D'
			        -- Khu vực IV - Bắc Trung Bộ
			        WHEN pos_city IN ('Thanh Hóa', 'Nghệ An', 'Hà Tĩnh', 'Quảng Bình', 
			                           'Quảng Trị', 'Huế') THEN 'E'
			        -- Khu vực V - Nam Trung Bộ
			        WHEN pos_city IN ('Đà Nẵng', 'Quảng Nam', 'Quảng Ngãi', 'Bình Định', 
			                           'Phú Yên', 'Khánh Hòa', 'Ninh Thuận', 'Bình Thuận', 
			                           'Kon Tum', 'Gia Lai', 'Đắk Lắk', 'Đắk Nông', 
			                           'Lâm Đồng') THEN 'F'
			        -- Khu vực VI - Miền Tây Nam Bộ
			        WHEN pos_city IN ('Cần Thơ', 'Long An', 'Đồng Tháp', 'Tiền Giang', 
			                           'An Giang', 'Bến Tre', 'Vĩnh Long', 'Trà Vinh', 
			                           'Hậu Giang', 'Kiên Giang', 'Sóc Trăng', 'Bạc Liêu', 
			                           'Cà Mau') THEN 'G'
			        -- Khu vực VII - Miền Đông Nam Bộ
			        WHEN pos_city IN ('Hồ Chí Minh', 
			                           'Bà Rịa - Vũng Tàu', 'Bình Dương', 'Bình Phước', 
			                           'Đồng Nai', 'Tây Ninh') THEN 'H'
			        -- Đơn vị phòng ban - HEAD
			        ELSE 'A'
			   	end;
```

### Tạo index
Vì bảng **fact_kpi_month_raw_data** có hơn 3 triệu records nên việc đánh index sẽ giúp quá trình truy vấn được cải thiện rất nhiều. Sau khi xem xét và xác định các cột cần lọc nhiều, ta đánh index trên 3 cột như sau:
```sql
create index idx_month ON fact_kpi_month_raw_data (kpi_month);
create index idx_area ON fact_kpi_month_raw_data (area_code);
create index idx_bucket ON fact_kpi_month_raw_data (max_bucket);
```

### Tạo bảng lưu trữ output của Procedure và bảng ghi log
Việc tạo bảng ghi log là cần thiết để có thể dễ dàng xác định Procedure có thành công hay không, đồng thời xác định rõ lỗi (nếu có).
```sql
--Tạo bảng lưu trữ output
create table report_monthly (
	month_key int8 not null,
	index_id int not null,
	amount numeric not null,
	area_code varchar(50) not null);

--Tạo bảng log_tracking 
CREATE TABLE log_tracking (
	log_id serial4 PRIMARY KEY NOT NULL,
	procedure_name varchar(100) NULL,
	start_time timestamp NULL,
	end_time timestamp NULL,
	is_successful varchar(1) NULL,
	error_log varchar(4000) NULL,
	rec_created_dt timestamp NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Thay đổi cấu trúc bảng kpi_asm_data
Nhận thấy cấu trúc bảng **kpi_asm_data** rất khó để xử lí trong Procedure (thực hiện các câu lệnh mang tính động), do đó cần thay đổi cấu trúc hợp lý để dễ dàng thực hiện quá trình này. Cấu trúc sẽ thay đổi như sau:

	+ month_key: Giá trị trải đều từ tháng 1 tới tháng 12 của năm đó

  	+ area_code: Mã khu vực giữ nguyên

     	+ sale_name: Tên nhân viên

      	+ email: Email của nhân viên

       	+ load_to_new: Giá trị Load to new tương ứng cho tháng đó

 	+ psdn: Giá  trị phát sinh dư nợ cho tháng đó

  	+ app_aprv: Số lượng đơn vay được chấp nhận

   	+ app_in: Số lượng đơn vay đăng ký

    	+ app_rate: app_aprv/app_in

Chi tiết như sau:
```sql
create or replace procedure change_structure()
language plpgsql 
as $$
declare
	--Các biến ghi nhận thời gian hoàn thành procedure
	v_start_time TIMESTAMPTZ;
	v_end_time TIMESTAMPTZ;
	v_is_successful CHAR(1) := 'Y';
    	v_error_log TEXT;
    	v_log_id INT;
begin
	-- ---------------------
    -- THÔNG TIN NGƯỜI TẠO
    -- ---------------------
    -- Tên người tạo: Khánh Hòa
    -- Ngày tạo: 2024-Dec-20
        -- Mục đích : Thay đổi cấu trúc bảng kpi_asm_data

    -- ---------------------
    -- THÔNG TIN NGƯỜI CẬP NHẬT
    -- ---------------------
    -- Tên người cập nhật: 
    -- Ngày cập nhật: 
    -- Mục đích cập nhật: 

    -- ---------------------
    -- SUMMARY LUỒNG XỬ LÝ
    -- ---------------------
    -- Bước 1: Ghi nhận thời gian bắt đầu procedure
	-- Bước 2: Tạo bảng mới nếu chưa có
    -- Bước 3: Xóa dữ liệu trong bảng nếu đã tồn tại
    -- Bước 4: insert dữ liệu
    -- Bước 5: Xử lý ngoại lệ và ghi log (nếu cần)

    -- ---------------------
    -- CHI TIẾT CÁC BƯỚC
    -- ---------------------
	-- Bước 1: Ghi nhận thời gian bắt đầu procedure
	v_start_time := CURRENT_TIMESTAMP;
	insert into log_tracking(procedure_name, start_time, is_successful)
	values ('change_structure', v_start_time, v_is_successful)
	returning log_id into v_log_id;

	-- Bước 2: Tạo bảng mới nếu chưa có
	create table if not exists 
		kpi_asm_data_changed(
			month_key int,
			area_code varchar(50),
			sale_name varchar(1024),
			email varchar(1024),
			load_to_new int8,
			psdn int8,
			app_aprv int8,
			app_in int8,
			app_rate numeric);
		
	-- Bước 3: Xóa dữ liệu trong bảng nếu đã tồn tại
	truncate table kpi_asm_data_changed;

	-- Bước 4: insert dữ liệu
	insert into kpi_asm_data_changed (month_key, area_code, sale_name, email, load_to_new, psdn, app_aprv, app_in, app_rate)
	--Tháng 1
	select 
		202301 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, jan_ltn, jan_psdn, jan_app_aprv, jan_app_in, jan_app_rate
	from kpi_asm_data
	where jan_ltn is not null
	-- Thang 2
	union all
	select 
		202302 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, feb_ltn, feb_psdn, feb_app_aprv, feb_app_in, feb_app_rate
	from kpi_asm_data
	where feb_ltn is not null
	-- Thang 3
	union all
	select 
		202303 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, mar_ltn, mar_psdn, mar_app_aprv, mar_app_in, mar_app_rate
	from kpi_asm_data
	where mar_ltn is not null
	--Thang 4
	union all
	select 
		202304 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, apr_ltn, apr_psdn, apr_app_aprv, apr_app_in, apr_app_rate
	from kpi_asm_data
	where apr_ltn is not null
	--Thang 5
	union all
	select 
		202305 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, may_ltn, may_psdn, may_app_aprv, may_app_in, may_app_rate
	from kpi_asm_data
	where may_ltn is not null
	--Thang 6
	union all
	select 
		202306 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, jun_ltn, jun_psdn, jun_app_aprv, jun_app_in, jun_app_rate
	from kpi_asm_data
	where jun_ltn is not null
	-- Thang 7
	union all
	select 
		202307 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, july_ltn, july_psdn, july_app_aprv, july_app_in, july_app_rate
	from kpi_asm_data
	where july_ltn is not null
	--Thang 8
	union all
	select 
		202308 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, aug_ltn, aug_psdn, aug_app_aprv, aug_app_in, aug_app_rate
	from kpi_asm_data
	where aug_ltn is not null
	--Thang 9
	union all
	select 
		202309 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, sep_ltn, sep_psdn, sep_app_aprv, sep_app_in, sep_app_rate
	from kpi_asm_data
	where sep_ltn is not null
	--Thang 10
	union all
	select 
		202310 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, oct_ltn, oct_psdn, oct_app_aprv, oct_app_in, oct_app_rate
	from kpi_asm_data
	where oct_ltn is not null
	--Thang 11
	union all
	select 
		202311 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, nov_ltn, nov_psdn, nov_app_aprv, nov_app_in, nov_app_rate
	from kpi_asm_data
	where nov_ltn is not null
	--Thang 12
	union all
	select 
		202312 as month_key,
		case
			when area_name = 'Hội Sở' then 'A'
			when area_name = 'Đông Bắc Bộ' then 'B'
			when area_name = 'Tây Bắc Bộ' then 'C'
			when area_name = 'Đồng Bằng Sông Hồng' then 'D'
			when area_name = 'Bắc Trung Bộ' then 'E'
			when area_name = 'Nam Trung Bộ' then 'F'
			when area_name = 'Tây Nam Bộ' then 'G'
			else 'H'
		end as area_code,
		sale_name, email, dec_ltn, dec_psdn, dec_app_aprv, dec_app_in, dec_app_rate
	from kpi_asm_data
	where dec_ltn is not null;
	--Bước 5: Ghi nhận thời gian kết thúc và ghi exception
	v_end_time := NOW();
		--+update table with end time
	UPDATE log_tracking
    SET end_time = v_end_time,
        is_successful = v_is_successful
    WHERE log_id = v_log_id;
   	--address exception and notice that procedure isn't completed
	exception
		when others then
			v_end_time := NOW();
       	 	v_is_successful := 'N';
        	v_error_log := SQLERRM;
		UPDATE log_tracking
        SET end_time = v_end_time,
            is_successful = v_is_successful,
            error_log = v_error_log
        WHERE log_id = v_log_id;
end;
$$;
--Goi procedure
call change_structure()
```

### Tạo Procedure 

Sau khi xử lý dữ liệu đầu vào, ta tiến hành tạo Procedure chính để khai báo các chỉ tiêu theo tháng. Chi tiết như sau:

```sql
CREATE OR REPLACE PROCEDURE final_project.report_asm_ranking(IN monthkey_input integer DEFAULT NULL::integer)
 LANGUAGE plpgsql
AS $procedure$
declare 
	--Các biến khai báo cần phải tổng hợp
	vmonthkey int;
	current_month int:=extract(month from current_Date)::int;
	current_year int:=extract(year from current_Date)::int;
	--Các biến ghi nhận thời gian hoàn thành procedure
   	v_start_time TIMESTAMPTZ;
    v_end_time TIMESTAMPTZ;
    v_is_successful CHAR(1) := 'Y';
    v_error_log TEXT;
    v_log_id INT;
BEGIN
    -- ---------------------
    -- THÔNG TIN NGƯỜI TẠO
    -- ---------------------
    -- Tên người tạo: Khánh Hòa
    -- Ngày tạo: 2024-Oct-28
        -- Mục đích : Tổng hợp các tiêu chí nguồn vốn và sử dụng vốn theo ngày

    -- ---------------------
    -- THÔNG TIN NGƯỜI CẬP NHẬT
    -- ---------------------
    -- Tên người cập nhật: 
    -- Ngày cập nhật: 
    -- Mục đích cập nhật: 

    -- ---------------------
    -- SUMMARY LUỒNG XỬ LÝ
    -- ---------------------
    -- Bước 1: Ghi nhận thời gian bắt đầu procedure
	-- Bước 2: Kiểm tra nếu tháng truyền vào là null sẽ lấy vmonthkey = Tháng hiện tại - 1
    -- Bước 3: xoá dữ liệu report_monthly tại tháng vmonthkey
    -- Bước 4: insert dữ liệu theo từng tiêu chí vào bảng
    -- Bước 5: Xử lý ngoại lệ và ghi log (nếu cần)

    -- ---------------------
    -- CHI TIẾT CÁC BƯỚC
    -- ---------------------

    -- Bước 1: Ghi nhận thời gian bắt đầu procedure
	v_start_time := CURRENT_TIMESTAMP;
	insert into log_tracking(procedure_name, start_time, is_successful)
	values ('report_asm_ranking', v_start_time, v_is_successful)
	returning log_id into v_log_id;

	-- Bước 2: Kiểm tra nếu tháng truyền vào là null sẽ lấy vmonthkey = Tháng hiện tại - 1
	if monthkey_input is null and current_month = 1
		then vmonthkey := (current_year - 1)*100 + 12;
	elseif monthkey_input is null and current_month != 1
		then vmonthkey := current_year*100 + current_month - 1;
	else vmonthkey := monthkey_input;
	end if;

	-- Bước 3: xoá dữ liệu report_monthly tại tháng vmonthkey
	delete from report_monthly where month_key = vmonthkey;

	-- Bước 4: insert dữ liệu theo từng tiêu chí vào bảng
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		-- Lãi trong hạn
		select 
			a.month_key,
			a.index_id,
			-- Lãi trong hạn = lãi trong hạn của riêng khu vực + phần lãi trong hạn của HEAD được chia cho từng khu vực theo tỷ lệ dư nợ của từng khu vực 
			coalesce(a.lai_trong_han + b.dnck_rate_n1*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 3 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 3
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			    area_code,
				sum(outstanding_principal)/x.avg_dnck_n1 as dnck_rate_n1
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n1
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey 
		    		and (max_bucket = 1 or max_bucket is null) 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1 
	    			and fkmrd2.kpi_month <= vmonthkey 
	    			and (max_bucket = 1 or max_bucket is null)
			    group by
			    	x.avg_dnck_n1,
			    	area_code
			   		) b
		on a.area_code = b.area_code
		-- Lãi quá hạn
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_qua_han + b.dnck_rate_n2*(
											select 
												sum(amount)  as lai_qua_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
												and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
											 	and index_id = 4 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_qua_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_qua_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 4
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			   area_code,
				sum(outstanding_principal)/x.avg_dnck_n2 as dnck_rate_n2
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n2
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
					kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey  
		    		and max_bucket = 2
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1 
	    			and fkmrd2.kpi_month <= vmonthkey 
	    			and fkmrd2.max_bucket = 2
			    group by
			    	x.avg_dnck_n2,
			    	area_code
			   		) b
		on a.area_code = b.area_code
		--Phí bảo hiểm
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_qua_han + b.psdn_rate*(
											select 
												sum(amount)  as lai_qua_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 5 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_qua_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_qua_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 5
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			    area_code,
				sum(psdn)/x.avg_psdn as psdn_rate
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(psdn) as avg_psdn
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1 
	    			and fkmrd2.kpi_month <= vmonthkey
			    group by
			    	x.avg_psdn,
			    	area_code
			   		) b
		on a.area_code = b.area_code
		--Phí tăng hạn mức
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.dnck_rate_n1*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 6 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 6
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			    area_code,
				sum(outstanding_principal)/x.avg_dnck_n1 as dnck_rate_n1
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n1
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey 
		    		and (max_bucket = 1 or max_bucket is null) 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1 
	    			and fkmrd2.kpi_month <= vmonthkey 
	    			and (max_bucket = 1 or max_bucket is null)
			    group by
			    	x.avg_dnck_n1,
			    	area_code
			   		) b
		on a.area_code = b.area_code
		--Phí thanh toán chậm, thu từ ngoại bảng, khác…
		union all	
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.dnck_rate_n2_5*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 7 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 7
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
				area_code,
				sum(outstanding_principal)/x.avg_dnck_n2_5 as dnck_rate_n2_5
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n2_5
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey and max_bucket >= 2
	    		) x
	    		on
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1 
	    			and fkmrd2.kpi_month <= vmonthkey and max_bucket >= 2
			    group by
			    	x.avg_dnck_n2_5,
			    	area_code
			   		) b
		on a.area_code = b.area_code;
		--Thu nhập từ hoạt động thẻ, lấy tổng các chỉ số ở 3,4,5,6,7 đã nhập vào report_monthly ở trên
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			2 as index_id,
			sum
				(
				case 
					when index_id in (3,4,5,6,7) then amount
				end
				) as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		--Chi phí vốn CCTG
		union all
		select 
			b.month_key,
			a.index_id,
			b.dnck_rate*a.phi_cctg/1000000 as amount,
			b.area_code
		from
			(
			select 
			 	vmonthkey as month_key,
			    area_code,
				sum(outstanding_principal)/x.avg_dnck as dnck_rate
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey
	    		) x
	    		on 
					fkmrd2.kpi_month >= (vmonthkey/100) + 1 
	    			and fkmrd2.kpi_month <= vmonthkey
			    group by
			    	x.avg_dnck,
			    	area_code
			   		) b
			left join 
			(
				select 
					vmonthkey as month_key,
					index_id,
					sum(amount)  as phi_cctg,
					area_code
				 from fact_txn_month_raw_data ftmrd2
				 where 
				 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
				 	and index_id = 12
				 group by
				 	index_id,
				 	area_code
			 ) a
		on 1=1
		union all
				select 
					vmonthkey as month_key,
					index_id,
					sum(amount)/1000000  as phi_cctg,
					area_code
				 from fact_txn_month_raw_data ftmrd2
				 where
				 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
				 	and index_id = 12
				 group by
				 	index_id,
				 	area_code
		-- Chi phí vốn TT1
		union all
		select 
			b.month_key,
			11 as index_id,
			coalesce(b.dnck_rate*a.phi_cctg,0)/1000000 as amount,
			b.area_code
		from
			(
			select 
			 	vmonthkey as month_key,
			    area_code,
				sum(outstanding_principal)/x.avg_dnck as dnck_rate
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck
		    	from fact_kpi_month_raw_data fkmrd 
		    	where
		    		kpi_month >= (vmonthkey/100) + 1 
		    		and kpi_month <= vmonthkey 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1
	    			and fkmrd2.kpi_month <= vmonthkey 
			    group by
			    	x.avg_dnck,
			    	area_code
			   		) b
			left join 
			(
				select 
					vmonthkey as month_key,
					index_id,
					sum(amount)  as phi_cctg,
					area_code
				 from fact_txn_month_raw_data ftmrd2
				 where 
				 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
				 	and index_id = 11
				 group by
				 	index_id,
				 	area_code
			 ) a
		on 1=1
		union all
				select 
					vmonthkey as month_key,
					11 as index_id,
					sum(amount)/1000000  as phi_cctg,
					'A' as area_code
				 from fact_txn_month_raw_data ftmrd2
				 where 
				 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
				 	and index_id = 11
				 group by
				 	index_id,
				 	area_code
		--Chi phí vốn TT2
		union all
		select 
			b.month_key,
			a.index_id,
			b.dnck_rate*a.phi_cctg/1000000 as amount,
			b.area_code
		from
			(
			select 
			 	vmonthkey as month_key,
			    area_code,
				sum(outstanding_principal)/x.avg_dnck as dnck_rate
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck
		    	from fact_kpi_month_raw_data fkmrd 
		    	where
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1
	    			and fkmrd2.kpi_month <= vmonthkey 
			    group by
			    	x.avg_dnck,
			    	area_code
			   		) b
			left join 
			(
				select 
					vmonthkey as month_key,
					index_id,
					sum(amount)  as phi_cctg,
					area_code
				 from fact_txn_month_raw_data ftmrd2
				 where 
				 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
				 	and index_id = 10
				 group by
				 	index_id,
				 	area_code
			 ) a
		on 1=1
		union all
				select 
					vmonthkey as month_key,
					index_id,
					sum(amount)/1000000  as phi_cctg,
					area_code
				 from fact_txn_month_raw_data ftmrd2
				 where 
				 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
				 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
				 	and index_id = 10
				 group by
				 	index_id,
				 	area_code;
		--Chi phí thuần KDV lấy tổng các index 9,10,11,12 đã thêm vào report_monthly ở trên
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			8 as index_id,
			sum
				(
				case 
					when index_id in (9,10,11,12) then amount
				end
				) as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		--CP hoa hồng
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.dnck_rate_n1*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 17 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 17
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			   area_code,
				sum(outstanding_principal)/x.avg_dnck_n1 as dnck_rate_n1
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n1
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1
	    			and fkmrd2.kpi_month <= vmonthkey
			    group by
			    	x.avg_dnck_n1,
			    	area_code
			   		) b
		on a.area_code = b.area_code
		--CP thuần KD khác
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.dnck_rate_n1*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 18 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 18
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			   area_code,
				sum(outstanding_principal)/x.avg_dnck_n1 as dnck_rate_n1
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n1
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1
	    			and fkmrd2.kpi_month <= vmonthkey
			    group by
			    	x.avg_dnck_n1,
			    	area_code
			   		) b
		on a.area_code = b.area_code
		--DT kinh doanh
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.dnck_rate_n1*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 16 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 16
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
			   area_code,
				sum(outstanding_principal)/x.avg_dnck_n1 as dnck_rate_n1
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(outstanding_principal) as avg_dnck_n1
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey 
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1
	    			and fkmrd2.kpi_month <= vmonthkey
			    group by
			    	x.avg_dnck_n1,
			    	area_code
			   		) b
		on a.area_code = b.area_code;
		--Chi phí thuần hoạt động khác
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			13 as index_id,
			sum
				(
				case 
					when index_id in (14,15,16,17,18,19) then amount
				end
				) as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code;
		--Tổng thu nhập hoạt động là tổng các index 2,8,13
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			20 as index_id,
			sum
				(
				case 
					when index_id in (2,8,13) then amount
				end
				) as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		-- CP thuế, phí
		union all
		select 
			a.month_key,
			22 as index_id,
			coalesce(a.lai_trong_han + b.slnv_rate*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 22 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 22
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	month_key,
			 	area_code,
				count(1)::numeric/x.total_slnv as slnv_rate
			from kpi_asm_data_changed a
			join 
			    (
				select 
		    		count(1) as total_slnv 
		    	from kpi_asm_data_changed 
		    	where month_key = vmonthkey
	    		) x
	    		on month_key = vmonthkey
			    group by
			    	a.month_key,
			    	x.total_slnv,
			    	a.area_code
			   		) b
		on a.area_code = b.area_code
		--CP nhân viên
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.slnv_rate*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 23 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 23
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	month_key,
			 	area_code,
				count(1)::numeric/x.total_slnv as slnv_rate
			from kpi_asm_data_changed a
			join 
			    (
				select 
		    		count(1) as total_slnv 
		    	from kpi_asm_data_changed 
		    	where month_key = vmonthkey
	    		) x
	    		on month_key = vmonthkey
			    group by
			    	a.month_key,
			    	x.total_slnv,
			    	a.area_code
			   		) b
		on a.area_code = b.area_code
		--CP quản lý
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.slnv_rate*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 24 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 24
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	month_key,
			 	area_code,
				count(1)::numeric/x.total_slnv as slnv_rate
			from kpi_asm_data_changed a
			join 
			    (
				select 
		    		count(1) as total_slnv 
		    	from kpi_asm_data_changed 
		    	where month_key = vmonthkey
	    		) x
	    		on month_key = vmonthkey
			    group by
			    	a.month_key,
			    	x.total_slnv,
			    	a.area_code
			   		) b
		on a.area_code = b.area_code
		--CP Tài sản
		union all
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.slnv_rate*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 25 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 25
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	month_key,
			 	area_code,
				count(1)::numeric/x.total_slnv as slnv_rate
			from kpi_asm_data_changed a
			join 
			    (
				select 
		    		count(1) as total_slnv 
		    	from kpi_asm_data_changed 
		    	where month_key = vmonthkey
	    		) x
	    		on month_key = vmonthkey
			    group by
			    	a.month_key,
			    	x.total_slnv,
			    	a.area_code
			   		) b
		on a.area_code = b.area_code;
	
		--Tổng chi phí là tổng các index 22,23,24,25
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			21 as index_id,
			sum
				(
				case 
					when index_id in (22,23,24,25) then amount
				end
				) as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
	
		--CP dự phòng
	union all	
		select 
			a.month_key,
			a.index_id,
			coalesce(a.lai_trong_han + b.dnck_rate_n2_5*(
											select 
												sum(amount)  as lai_trong_han
											 from fact_txn_month_raw_data ftmrd2
											 where 
											 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
											 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey 
											 	and index_id = 26 and split_part(analysis_code, '.', 4) = '00'
										 	),a.lai_trong_han)/1000000 as amount,
			a.area_code 
		from 
		(
			select 
				vmonthkey as month_key,
				index_id,
				sum(amount)  as lai_trong_han,
				area_code
			 from fact_txn_month_raw_data ftmrd2
			 where 
			 	extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int >= (vmonthkey/100) + 1
			 	and extract(year from transaction_date)::int*100 + extract(month from transaction_date)::int <= vmonthkey
			 	and index_id = 26
			 group by
			 	index_id,
			 	area_code
		 ) a
		left join
			(
			select 
			 	vmonthkey as month_key,
				area_code,
				(sum(case when max_bucket >= 2 then outstanding_principal end) + coalesce(sum(case when write_off_month <= vmonthkey and write_off_month >= 202301 then write_off_balance_principal end),0))/x.avg_dnck_n2_5 as dnck_rate_n2_5
			from fact_kpi_month_raw_data fkmrd2 
			join 
			    (
				select 
		    		sum(case when max_bucket >= 2 then outstanding_principal end) + coalesce(sum(case when write_off_month <= vmonthkey and write_off_month >= 202301 then write_off_balance_principal end),0) as avg_dnck_n2_5
		    	from fact_kpi_month_raw_data fkmrd 
		    	where 
		    		kpi_month >= (vmonthkey/100) + 1
		    		and kpi_month <= vmonthkey
	    		) x
	    		on 
	    			fkmrd2.kpi_month >= (vmonthkey/100) + 1
	    			and fkmrd2.kpi_month <= vmonthkey
			    group by
			    	x.avg_dnck_n2_5,
			    	area_code
			   		) b
		on a.area_code = b.area_code;
		--Lợi nhuận trước thuế là tổng các mã 20,21,26
		insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			1 as index_id,
			sum
				(
				case 
					when index_id in (20,21,26) then amount
				end
				) as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		--Số lượng nhân sự
	union all
		select 
			month_key,
			27 as index_id,
			count(1) as amount,
			area_code
		from kpi_asm_data_changed 
		where month_key = vmonthkey 
		group by month_key,area_code
		union all
		select
			month_key,
			27 as index_id,
	    	count(1) as total_slnv,
	    	'A' as area_code
	    from kpi_asm_data_changed
	    where month_key = vmonthkey
	    group by month_key;
		--CIR là index 21 chia 20
	insert into report_monthly  
		(month_key, index_id, amount, area_code)
		select 
			vmonthkey as month_key,
			29 as index_id,
			-sum
				(
				case 
					when index_id in (21) then amount
				end
				)*100::float8/
			sum
				(
				case 
					when index_id in (20) then amount
				end
				)as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		--Margin là index 1 chia cho tổng 2,9,14,15,16
	union all
		select 
			vmonthkey as month_key,
			30 as index_id,
			sum
				(
				case 
					when index_id in (1) then amount
				end
				)*100::float8/
			sum
				(
				case 
					when index_id in (2,9,14,15,16) then amount
				end
				)as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		--Hiệu suất trên/vốn (%) là 1 chia cho 8
	union all
		select 
			vmonthkey as month_key,
			31 as index_id,
			-sum
				(
				case 
					when index_id in (1) then amount
				end
				)*100::float8/
			sum
				(
				case 
					when index_id in (8) then amount
				end
				)as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code
		--Hiệu suất BQ/ Nhân sự là 1 chia 27
	union all
		select 
			vmonthkey as month_key,
			32 as index_id,
			sum
				(
				case 
					when index_id in (1) then amount
				end
				)::float8/
			sum
				(
				case 
					when index_id in (27) then amount
				end
				)as amount,
			area_code
		from report_monthly 
		where month_key = vmonthkey
		group by area_code;
	
   	--Bước 5: Ghi nhận thời gian kết thúc và ghi exception
	v_end_time := CURRENT_TIMESTAMP;
		--+update table with end time
	UPDATE log_tracking
    SET end_time = v_end_time,
        is_successful = v_is_successful
    WHERE log_id = v_log_id;
   	--address exception and notice that procedure isn't completed
	exception
		when others then
			v_end_time := CURRENT_TIMESTAMP;
       	 	v_is_successful := 'N';
        	v_error_log := SQLERRM;
		UPDATE log_tracking
        SET end_time = v_end_time,
            is_successful = v_is_successful,
            error_log = v_error_log
        WHERE log_id = v_log_id;
end;
$procedure$
;
--Gọi Procedure với tháng 2 giả định
call report_asm_ranking(202302);
```

### Truy vấn output

Output cần có 1 bảng báo cáo kết quả kinh doanh các khu vực và 1 bảng báo cáo xếp hạng nhân sự quản lý chi nhánh.
```sql
-- Bảng báo cáo kết quả kinh doanh các khu vực
create or replace view 
	report_monthly_view as
		select 
			d.index_id,
			d.index_name,
			coalesce(max(case when area_code = 'A' then amount end),0) as A,
			coalesce(max(case when area_code = 'B' then amount end),0) as B,
			coalesce(max(case when area_code = 'C' then amount end),0) as c,
			coalesce(max(case when area_code = 'D' then amount end),0) as d,
			coalesce(max(case when area_code = 'E' then amount end),0) as e,
			coalesce(max(case when area_code = 'F' then amount end),0) as f,
			coalesce(max(case when area_code = 'G' then amount end),0) as g,
			coalesce(max(case when area_code = 'H' then amount end),0) as h,
			coalesce(sum(case when area_code != 'A' then amount end),0) as total
		from dim_indexes d
		left join report_monthly rm
		on d.index_id = rm.index_id and rm.month_key = 202302
		group by d.index_name,d.index_id 
		order by d.index_id;

--Với bảng xếp hạng nhân sự, cần tạo thêm 1 bảng khác bao gồm các chỉ tiêu để đánh giá
create table report_kpi (
	month_key int8,
	area_code varchar(1024),
	area_name varchar(1024),
	email varchar(1024),
	tong_diem int8,
	rank_final int8,
	ltn_avg int8,
	rank_ltn_avg int8,
	psdn_avg int8,
	rank_psdn_avg int8,
	approval_rate_avg int8,
	rank_approval_rate_avg int8,
	npl_bef_wo int8,
	rank_npl_bef_wo int8,
	diem_quy_mo int8,
	rank_ptkd int8,
	cir int8,
	rank_cir int8,
	margin int8,
	rank_margin int8,
	hs_von int8,
	rank_hs_von int8,
	hsbq_nhan_su int8,
	rank_hsbq_nhan_su int8,
	diem_fin int8,
	rank_fin int8
);
--Tạo procedure để nhập dữ liệu vào bảng vừa tạo
create or replace procedure kpi_monthly(monthkey_input int)
language plpgsql 
as $$
begin 
	--Xóa dữ liệu tháng cần khai báo hiện tại
	delete from report_kpi where month_key = monthkey_input;
	--Nhập dữ liệu vào bảng
	insert into report_kpi 
		(month_key,area_code,area_name,email,tong_diem,rank_final,ltn_avg,rank_ltn_avg,psdn_avg,rank_psdn_avg,
		approval_rate_avg,rank_approval_rate_avg,npl_bef_wo,rank_npl_bef_wo,diem_quy_mo,rank_ptkd,cir,rank_cir,
		margin,rank_margin,hs_von,rank_hs_von,hsbq_nhan_su,rank_hsbq_nhan_su,diem_fin,rank_fin)
	select 
		month_key, area_code, area_name, email,
		diem_quy_mo + diem_fin as tong_diem,
		row_number() over(order by diem_quy_mo + diem_fin asc) as rank_final,
		ltn_avg, rank_ltn_avg, psdn_avg, rank_psdn_avg, 
		approval_rate_avg, rank_approval_rate_avg,npl_bef_wo, rank_npl_bef_wo, diem_quy_mo,
		rank_ptkd, cir, rank_cir, margin, rank_margin, hs_von, rank_hs_von, hsbq_nhan_su, rank_hsbq_nhan_su,diem_fin,rank_fin
	from
	(
	select 
		month_key,
		area_code,
		case 
			when area_code = 'B' then 'Đông Bắc Bộ'
			when area_code = 'C' then 'Tây Bắc Bộ'
			when area_code = 'D' then 'Đồng Bằng Sông Hồng'
			when area_code = 'E' then 'Bắc Trung Bộ'
			when area_code = 'F' then 'Nam Trung Bộ'
			when area_code = 'G' then 'Tây Nam Bộ'
			else 'Đông Nam Bộ'
		end as area_name, email,
		ltn_avg, rank_ltn_avg, psdn_avg, rank_psdn_avg, approval_rate_avg, rank_approval_rate_avg,npl_bef_wo, rank_npl_bef_wo,
		rank_ltn_avg + rank_psdn_avg + rank_approval_rate_avg + rank_npl_bef_wo as diem_quy_mo,
		row_number() over(order by rank_ltn_avg + rank_psdn_avg + rank_approval_rate_avg + rank_npl_bef_wo asc) as rank_ptkd,
		cir, rank_cir, margin, rank_margin, hs_von, rank_hs_von, hsbq_nhan_su, rank_hsbq_nhan_su,
		rank_cir + rank_margin + rank_hs_von + rank_hsbq_nhan_su as diem_fin,
		row_number() over(order by rank_cir + rank_margin + rank_hs_von + rank_hsbq_nhan_su asc) as rank_fin	 
	from
		(
		select 
			a.month_key,
			a.area_code,
			a.email,
			a.ltn_avg,
			dense_rank() over(order by a.ltn_avg desc) as rank_ltn_avg,
			a.psdn_avg,
			dense_rank() over(order by a.psdn_avg desc) as rank_psdn_avg,
			a.approval_rate_avg,
			dense_rank() over(order by a.approval_rate_avg desc) as rank_approval_rate_avg,
			a.npl_bef_wo,
			dense_rank() over(order by a.npl_bef_wo asc) as rank_npl_bef_wo,
			b.cir,
			dense_rank() over(order by b.cir asc) as rank_cir,
			b.margin,
			dense_rank() over(order by b.margin desc) as rank_margin,
			b.hs_von,
			dense_rank() over(order by b.hs_von desc) as rank_hs_von,
			b.hsbq_nhan_su,
			dense_rank() over(order by b.hsbq_nhan_su desc) as rank_hsbq_nhan_su
		from 
			(-- Các chỉ tiêu Load to new, psdn, app_rate đã có sẵn ở bảng kpi_asm_data_changed
			select 
				monthkey_input as month_key,
				kadc.area_code,
				email,
				avg(load_to_new) as ltn_avg,
				avg(psdn) as psdn_avg,
				avg(app_rate) as approval_rate_avg,
				npl_bef_wo
			from kpi_asm_data_changed kadc
			join 
			(-- chỉ tiêu NPL
			select 
				area_code,
				(sum(case when max_bucket >= 3 then outstanding_principal end) 
				+ sum(case when write_off_month <= monthkey_input and write_off_month >= 202301 then write_off_balance_principal end))
				/(sum(outstanding_principal) 
				+ sum(case when write_off_month <= monthkey_input and write_off_month >= 202301 then write_off_balance_principal end)) as npl_bef_wo
			from fact_kpi_month_raw_data fkmrd 
			where kpi_month = monthkey_input
			group by area_code
			) x
			on kadc.area_code = x.area_code and month_key <= monthkey_input
			group by email, kadc.area_code, npl_bef_wo
			) a
		join 
			(-- Lấy các chỉ tiêu từ kết quả bên trên, với các chỉ tiêu theo khu vực này thì giá trị của nhân sự = giá trị của khu vực đó
			select 
				area_code,
				-sum(case when index_id in (21) then amount end)
				/sum(case when index_id in (20) then amount end) as cir,
				sum(case when index_id = 1 then amount end)
				/sum(case when index_id in (2,9,14,15,16) then amount end) as margin,
				-sum(case when index_id = 1 then amount end)
				/sum(case when index_id in (8) then amount end) as hs_von,
				sum(case when index_id = 1 then amount end)
				/sum(case when index_id in (27) then amount end) as hsbq_nhan_su
			from report_monthly rm2 
			group by area_code
			) b
		on a.area_code = b.area_code
		));
end; 
$$
--Gọi Procedure
call kpi_monthly(202302)
```

### Kéo output sang Power BI và tiến hành phân tích

Thực hiện kéo view/table sang Power BI bằng direct query hoặc import. Kết quả phân tích có thể xem dưới đây:

[Kết quả phân tích trên Power BI](https://app.powerbi.com/view?r=eyJrIjoiNDZmNWNlMWEtZDdjOC00OWMzLThiODctZmM5MDdlMjA0MTY4IiwidCI6ImQ2ZDEzZTBlLTdjYTAtNDNkNC05OTY1LTQyZDM4ZWU1M2RkYSIsImMiOjEwfQ%3D%3D)

