# BÁO CÁO THỰC HÀNH KIỂM THỬ TỰ ĐỘNG BẰNG SELENIUM

## 2. Giới thiệu bài tập
- **Công cụ sử dụng:** Selenium WebDriver (Python)
- **Môi trường thực thi:** Google Colab (Trình duyệt Chrome Headless)
- **Website kiểm thử:** https://www.saucedemo.com/ (Trang thương mại điện tử thử nghiệm)
- **Mục tiêu:** Xây dựng kịch bản kiểm thử tự động cho 05 chức năng chính của website bao gồm: Đăng nhập thành công, Lọc sản phẩm, Thêm vào giỏ hàng, Đăng xuất và Kiểm tra thông báo lỗi khi đăng nhập thất bại.

---

## 3. Danh sách 05 Test Cases

| STT | Mã TC | Chức năng kiểm thử | Các bước thực hiện (Steps) | Kết quả mong đợi (Expected) | Trạng thái |
|:---:|:---:|---|---|---|:---:|
| **1** | **TC01** | Đăng nhập thành công | 1. Truy cập trang chủ.<br>2. Nhập username `standard_user` & password `secret_sauce`.<br>3. Nhấn nút Login. | Chuyển hướng thành công sang trang danh sách sản phẩm (`/inventory.html`). | **PASSED** |
| **2** | **TC02** | Lọc sản phẩm theo giá | 1. Mở bộ lọc sản phẩm.<br>2. Chọn kiểu lọc "Price (low to high)". | Danh sách sản phẩm được sắp xếp lại, sản phẩm đầu tiên có giá `$7.99`. | **PASSED** |
| **3** | **TC03** | Thêm sản phẩm vào giỏ hàng | 1. Nhấn nút "Add to cart" ở sản phẩm đầu tiên. | Biểu tượng giỏ hàng xuất hiện badge hiển thị số lượng là `1`. | **PASSED** |
| **4** | **TC04** | Đăng xuất | 1. Nhấn mở Menu bên góc trái.<br>2. Nhấn nút "Logout". | Tài khoản đăng xuất thành công và quay trở về màn hình đăng nhập. | **PASSED** |
| **5** | **TC05** | Đăng nhập thất bại | 1. Nhập username đúng & password sai (`wrong_pass`).<br>2. Nhấn nút Login. | Hệ thống chặn đăng nhập và hiển thị thông báo lỗi `Username and password do not match...`. | **PASSED** |

---

## 4. Mã nguồn kịch bản kiểm thử (Test Script)

Kịch bản kiểm thử được viết bằng ngôn ngữ Python với Selenium WebDriver:

```python
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

# 1. Cấu hình Chrome chạy trên môi trường Colab (Headless)
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome_options.binary_location = '/usr/bin/chromium-browser'

# 2. Khởi tạo Driver
service = Service('/usr/bin/chromedriver')
driver = webdriver.Chrome(service=service, options=chrome_options)
driver.implicitly_wait(10)

try:
    print("--- BẮT ĐẦU CHẠY 05 TEST CASES ---")

    # TC01: Đăng nhập thành công
    driver.get("[https://www.saucedemo.com/](https://www.saucedemo.com/)")
    driver.find_element(By.ID, "user-name").send_keys("standard_user")
    driver.find_element(By.ID, "password").send_keys("secret_sauce")
    driver.find_element(By.ID, "login-button").click()
    assert "inventory.html" in driver.current_url
    print("✅ TC01 - Đăng nhập thành công: PASSED")

    # TC02: Lọc sản phẩm theo giá
    sort_dropdown = driver.find_element(By.CLASS_NAME, "product_sort_container")
    sort_dropdown.click()
    driver.find_element(By.XPATH, "//option[@value='lohi']").click()
    time.sleep(1)
    first_price = driver.find_element(By.CLASS_NAME, "inventory_item_price").text
    assert first_price == "$7.99"
    print("✅ TC02 - Lọc sản phẩm theo giá: PASSED")

    # TC03: Thêm sản phẩm vào giỏ hàng
    add_btn = driver.find_element(By.CSS_SELECTOR, "button[id^='add-to-cart']")
    add_btn.click()
    time.sleep(1)
    cart_badge = driver.find_element(By.CLASS_NAME, "shopping_cart_badge").text
    assert cart_badge == "1"
    print("✅ TC03 - Thêm vào giỏ hàng: PASSED")

    # TC04: Đăng xuất
    driver.find_element(By.ID, "react-burger-menu-btn").click()
    time.sleep(1)
    driver.find_element(By.ID, "logout_sidebar_link").click()
    assert driver.find_element(By.ID, "login-button").is_displayed()
    print("✅ TC04 - Đăng xuất: PASSED")

    # TC05: Đăng nhập thất bại
    driver.find_element(By.ID, "user-name").send_keys("standard_user")
    driver.find_element(By.ID, "password").send_keys("wrong_pass")
    driver.find_element(By.ID, "login-button").click()
    error_msg = driver.find_element(By.XPATH, "//h3[@data-test='error']").text
    assert "Username and password do not match" in error_msg
    print("✅ TC05 - Đăng nhập thất bại: PASSED")

finally:
    driver.quit()
    print("--- HOÀN THÀNH TOÀN BỘ TEST CASES ---")
