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

