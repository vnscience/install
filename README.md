### **Lưu ý quan trọng**

1.  **Chạy với quyền Quản trị (Administrator):** Tập lệnh này cần được chạy trong một cửa sổ PowerShell với quyền quản trị. Nhấp chuột phải vào biểu tượng PowerShell và chọn "Run as administrator".
2.  **Winget:** Tập lệnh này sử dụng `winget`, trình quản lý gói tích hợp sẵn của Windows. Nó hoạt động trên Windows 10 (phiên bản 1809 trở lên) và Windows 11.
3.  **Tùy chỉnh:** Bạn có thể dễ dàng tùy chỉnh tập lệnh. Nếu bạn không muốn cài đặt một phần mềm cụ thể, chỉ cần thêm dấu thăng (`#`) vào đầu dòng tương ứng để bỏ qua nó.
4.  **Phần mềm thương mại & phiên bản cũ:**
      * Đối với các phần mềm thương mại yêu cầu giấy phép (như VMWare, MYOB, NetSupport), tập lệnh sẽ cung cấp một hàm mẫu. Bạn cần tải xuống tệp cài đặt từ trang web của nhà cung cấp và đặt nó vào cùng một thư mục với tập lệnh này.
      * Nhiều thành phần trong danh sách của bạn là các gói phụ thuộc cũ của Visual Studio 2015 và các SDK. Cách tiếp cận hiện đại và được khuyến nghị là cài đặt phiên bản Visual Studio mới nhất (ví dụ: 2022) với các workload cần thiết, vì nó sẽ tự động quản lý các SDK và thành phần này. Tập lệnh sẽ cài đặt Visual Studio 2022 Community với các workload phổ biến.
      * Tương tự, các thành phần Office được cài đặt tốt nhất thông qua Công cụ triển khai Office (Office Deployment Tool).

-----

### **Tập lệnh PowerShell**

Lưu mã dưới đây vào một tệp có tên `Install-Software.ps1`.

```powershell
<#
.SYNOPSIS
    Tu dong cai dat mot danh sach cac phan mem phat trien va cong cu voi giao dien tien do.

.DESCRIPTION
    Tap lenh nay se khoi chay cac trinh cai dat o che do tuong tac de ban co the theo doi tien do tai xuong va cai dat.
    
    LUU Y: Chay tap lenh nay voi quyen Quan tri vien (Run as Administrator).
    Tuy chinh cac phan mem duoc cai dat bang cach them/xoa dau thang (#) o dau moi dong.

.AUTHOR
    Dua tren yeu cau cua nguoi dung.

.VERSION
    2.0
#>

#================================================================================
# HAM TRO GIUP
#================================================================================

# Ham de cai dat phan mem thuong mai hoac phan mem yeu cau tep cai dat cuc bo
Function Install-LocalOrCommercial-Software {
    param(
        [string]$SoftwareName,
        [string]$InstallerName,
        [string]$InstallerArgs # Tham so nay co the de trong de chay voi giao dien
    )

    $installerPath = Join-Path $PScriptRoot $InstallerName
    
    if (Test-Path $installerPath) {
        Write-Host "Bat dau cai dat $SoftwareName..." -ForegroundColor Green
        Write-Host "Cua so cai dat cua $SoftwareName se hien len. Vui long lam theo cac buoc de hoan tat." -ForegroundColor Yellow
        try {
            Start-Process -FilePath $installerPath -ArgumentList $InstallerArgs -Wait -PassThru -ErrorAction Stop
            Write-Host "$SoftwareName da duoc cai dat thanh cong." -ForegroundColor Green
        }
        catch {
            Write-Error "Loi khi cai dat $SoftwareName. Chi tiet: $_"
        }
    }
    else {
        Write-Warning "Khong tim thay tep cai dat cho $SoftwareName tai '$installerPath'. Vui long tai ve va dat vao cung thu muc voi tap lenh."
    }
}

# Ham de cai dat Visual Studio 2022 Community voi giao dien tien do
Function Install-VisualStudio {
    Write-Host "Bat dau cai dat Visual Studio 2022 Community..." -ForegroundColor Cyan
    Write-Host "Trinh cai dat Visual Studio se khoi chay de ban co the theo doi tien do." -ForegroundColor Yellow
    try {
        $vsInstallerUrl = "https://aka.ms/vs/17/release/vs_Community.exe"
        $vsInstallerPath = Join-Path $env:TEMP "vs_Community.exe"
        
        Invoke-WebRequest -Uri $vsInstallerUrl -OutFile $vsInstallerPath
        
        # Them cac workload can thiet vao day.
        # De xem danh sach day du cac ID workload, hay truy cap: https://learn.microsoft.com/en-us/visualstudio/install/workload-and-component-ids
        $workloads = @(
            "--add Microsoft.VisualStudio.Workload.ManagedDesktop", # .NET desktop development
            "--add Microsoft.VisualStudio.Workload.NativeDesktop",   # Desktop development with C++
            "--add Microsoft.VisualStudio.Workload.NetWeb",        # ASP.NET and web development
            "--add Microsoft.VisualStudio.Workload.Azure",         # Azure development
            "--add Microsoft.VisualStudio.Workload.Data",          # Data storage and processing
            "--includeRecommended"
        )
        
        # Loai bo co --quiet de hien thi giao dien nguoi dung cua trinh cai dat
        $arguments = "$workloads --norestart"
        
        Start-Process -FilePath $vsInstallerPath -ArgumentList $arguments -Wait -PassThru
        Write-Host "Visual Studio 2022 Community da duoc cai dat." -ForegroundColor Green
    }
    catch {
        Write-Error "Loi khi cai dat Visual Studio. Chi tiet: $_"
    }
}

#================================================================================
# CAI DAT PHAN MEM
#================================================================================

Write-Host "Bat dau qua trinh cai dat phan mem. Vui long dam bao ban dang chay voi quyen Quan tri vien." -ForegroundColor Yellow
Write-Host "----------------------------------------------------------------"

# --- MOI TRUONG PHAT TRIEN & IDEs ---
Write-Host "Dang cai dat Moi truong phat trien & IDEs..." -ForegroundColor Cyan
# Cai dat Visual Studio voi giao dien tien do
Install-VisualStudio

# Cac lenh winget se tu dong hien thi thanh tien trinh trong PowerShell
winget install --id Microsoft.VisualStudioCode -e --accept-package-agreements # Visual Studio Code
winget install --id Oracle.Java.JDK.18 -e --accept-package-agreements         # Java SE Development Kit 18
winget install --id Oracle.Java.JDK.8 -e --accept-package-agreements          # Java SE Development Kit 8
winget install --id Python.Python.3.10 -e --accept-package-agreements         # Python 3.10
winget install --id Gluon.SceneBuilder -e --accept-package-agreements         # SceneBuilder
winget install --id Node.js -e --accept-package-agreements                    # Node.js

# --- CO SO DU LIEU ---
Write-Host "Dang cai dat Cong cu Co so du lieu..." -ForegroundColor Cyan
winget install --id Oracle.MySQL.Server -e --accept-package-agreements --scope machine # MySQL Server
winget install --id Oracle.MySQL.Workbench -e --accept-package-agreements             # MySQL Workbench
winget install --id Oracle.MySQL.Shell -e --accept-package-agreements                 # MySQL Shell
winget install --id Oracle.MySQL.Connector.ODBC -e --accept-package-agreements        # MySQL Connector/ODBC
winget install --id Oracle.MySQL.Connector.CPP -e --accept-package-agreements         # MySQL Connector C++
winget install --id Oracle.MySQL.Connector.NET -e --accept-package-agreements         # MySQL Connector Net
winget install --id Microsoft.SQLServer.2019.Developer -e --accept-package-agreements # SQL Server 2019 Developer Edition
winget install --id Microsoft.SQLServerManagementStudio -e --accept-package-agreements    # SQL Server Management Studio (SSMS)


# --- CONG CU & TIEN ICH ---
Write-Host "Dang cai dat Cong cu & Tien ich..." -ForegroundColor Cyan
winget install --id 7zip.7zip -e --accept-package-agreements                   # 7-Zip
winget install --id Git.Git -e --accept-package-agreements                     # Git
winget install --id Apache.OpenOffice -e --accept-package-agreements           # OpenOffice
winget install --id DevinCook.Flowgorithm -e --accept-package-agreements       # Flowgorithm
winget install --id Microsoft.Edge -e --accept-package-agreements              # Microsoft Edge (Thuong da duoc cai san)

# --- PHAN MEM THUONG MAI & YEU CAU TEP CAI DAT CUC BO ---
# Huong dan: Tai tep cai dat (.exe hoac .msi) va dat no vao cung thu muc voi tap lenh.
# De trong tham so -InstallerArgs de chay trinh cai dat voi giao dien do hoa day du.

Write-Host "Dang xu ly phan mem thuong mai (yeu cau tep cai dat cuc bo)..." -ForegroundColor Cyan

# Vi du cho VMware Workstation (de trong InstallerArgs de thay giao dien)
# Tai tep cai dat vi du: VMware-workstation-full-xx.x.x-xxxx.exe
# Install-LocalOrCommercial-Software -SoftwareName "VMware Workstation" -InstallerName "VMware-workstation-full-xx.x.x-xxxx.exe" -InstallerArgs ""

# Vi du cho MYOB AccountRight
# Install-LocalOrCommercial-Software -SoftwareName "MYOB AccountRight" -InstallerName "MYOB_AccountRight_2018.3.exe" -InstallerArgs ""

# Vi du cho NetSupport School
# Install-LocalOrCommercial-Software -SoftwareName "NetSupport School" -InstallerName "NetSupport_School_Tutor.exe" -InstallerArgs ""


# --- DRIVERS & PHAN MEM OEM ---
Write-Host "Dang cai dat Drivers & Phan mem OEM..." -ForegroundColor Cyan
winget install --id Dell.SupportAssist -e --accept-package-agreements         # Dell SupportAssist
winget install --id Intel.WirelessBluetooth -e --accept-package-agreements    # Intel Wireless Bluetooth
winget install --id Intel.ChipsetDeviceSoftware -e --accept-package-agreements# Intel Chipset Software

Write-Host "----------------------------------------------------------------"
Write-Host "Tat ca cac tac vu da hoan tat!" -ForegroundColor Green

```

### **Cách sử dụng**

1.  **Lưu tập lệnh:** Sao chép mã ở trên và lưu nó vào một tệp có tên `Install-Software.ps1` trên máy tính mới.
2.  **Tải xuống tệp cài đặt cần thiết:** Đối với các phần mềm như VMware, MYOB, bạn cần tải xuống tệp cài đặt từ trang web của họ và đặt chúng vào cùng thư mục với tệp `Install-Software.ps1`. Bạn cũng cần cập nhật tên tệp cài đặt trong tập lệnh.
3.  **Mở PowerShell với quyền Quản trị:**
      * Nhấn phím `Windows`.
      * Gõ "PowerShell".
      * Nhấp chuột phải vào "Windows PowerShell" và chọn "Run as administrator".
4.  **Cho phép chạy tập lệnh:** Theo mặc định, PowerShell có thể chặn việc chạy các tập lệnh. Chạy lệnh sau để cho phép:
    ```powershell
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
    ```
5.  **Điều hướng đến thư mục chứa tập lệnh:** Sử dụng lệnh `cd` để di chuyển đến thư mục bạn đã lưu tệp. Ví dụ, nếu bạn lưu nó trên Desktop:
    ```powershell
    cd $env:USERPROFILE\Desktop
    ```
6.  **Chạy tập lệnh:**
    ```powershell
    .\Install-Software.ps1
    ```

Tập lệnh sẽ tự động chạy và cài đặt từng phần mềm một. Quá trình này có thể mất khá nhiều thời gian tùy thuộc vào số lượng phần mềm và tốc độ mạng của bạn.


# LỖI THƯỜNG GẶP

`winget` không chạy được là một sự cố khá phổ biến. Dưới đây là các nguyên nhân và cách khắc phục chi tiết, xếp theo thứ tự từ dễ đến khó.

Bạn hãy thử từng bước một nhé.

-----

### \#\# 1. Kiểm tra phiên bản Windows 🔍

Đầu tiên, `winget` yêu cầu phiên bản Windows 10 từ **1809** trở lên hoặc **Windows 11**.

  * Nhấn tổ hợp phím `Windows` + `R`.
  * Gõ `winver` và nhấn Enter.
  * Một cửa sổ sẽ hiện ra, hãy kiểm tra dòng "Version". Nếu phiên bản của bạn cũ hơn **1809**, bạn cần phải cập nhật Windows trước.

-----

### \#\# 2. Cài đặt hoặc cập nhật "App Installer" từ Microsoft Store (Cách phổ biến nhất) 🛒

`winget` được phân phối thông qua một ứng dụng có tên là **App Installer** trên Microsoft Store. Việc `winget` không chạy thường là do ứng dụng này bị thiếu hoặc đã cũ.

1.  Mở **Microsoft Store**.
2.  Tìm kiếm "App Installer".
3.  Nếu bạn thấy nút **"Cập nhật" (Update)** hoặc **"Tải về" (Get)**, hãy nhấn vào đó.

Hoặc, bạn có thể truy cập trực tiếp vào trang của App Installer qua liên kết này và nhấn "Get in Store app" để mở ứng dụng trong Microsoft Store:

[**App Installer trên Microsoft Store**](https://www.google.com/search?q=https://apps.microsoft.com/store/detail/app-installer/9NBLGGH4NNS1)

Sau khi cài đặt hoặc cập nhật xong, hãy **khởi động lại PowerShell** và thử lại lệnh `winget`.

-----

### \#\# 3. Kiểm tra biến môi trường PATH ⚙️

Đôi khi App Installer đã được cài đặt nhưng đường dẫn đến `winget.exe` chưa được thêm vào biến môi trường PATH của hệ thống.

1.  Mở **PowerShell**.

2.  Chạy lệnh sau để kiểm tra xem đường dẫn `WindowsApps` có trong PATH không:

    ```powershell
    $env:Path -split ';' | Select-String 'WindowsApps'
    ```

    Nếu bạn thấy kết quả có chứa `Microsoft\WindowsApps`, nghĩa là đường dẫn đã đúng. Nếu không có kết quả nào, bạn cần thêm thủ công.

3.  **Cách thêm PATH thủ công:**

      * Mở **Start Menu**, gõ "Edit the system environment variables" và mở nó.
      * Trong cửa sổ System Properties, chọn **Environment Variables...**.
      * Trong mục "User variables for [Tên người dùng của bạn]", tìm và chọn biến **Path**, sau đó nhấn **Edit...**.
      * Nhấn **New** và dán đường dẫn sau:
        ```
        %USERPROFILE%\AppData\Local\Microsoft\WindowsApps
        ```
      * Nhấn **OK** ở tất cả các cửa sổ để lưu lại.
      * **Khởi động lại máy tính** hoặc ít nhất là khởi động lại PowerShell và thử lại.

-----

### \#\# 4. Cài đặt thủ công từ GitHub (Nếu Store không hoạt động) 📦

Nếu bạn không thể sử dụng Microsoft Store, bạn có thể tải và cài đặt `winget` trực tiếp từ kho mã nguồn của Microsoft trên GitHub.

1.  Truy cập trang phát hành chính thức: [**GitHub - winget-cli Releases**](https://github.com/microsoft/winget-cli/releases)
2.  Tìm phiên bản mới nhất (thường ở trên cùng và có nhãn "Latest").
3.  Tải xuống tệp có đuôi `.msixbundle`.
4.  Mở **PowerShell với quyền Quản trị (Administrator)**, điều hướng đến thư mục bạn vừa tải tệp về (ví dụ: `cd $env:USERPROFILE\Downloads`).
5.  Chạy lệnh sau (thay `AppName` bằng tên tệp bạn đã tải):
    ```powershell
    Add-AppxPackage -Path ".\AppName.msixbundle"
    ```
    Ví dụ: `Add-AppxPackage -Path ".\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle"`
6.  Khởi động lại PowerShell và kiểm tra lại `winget`.

-----

### \#\# 5. Kiểm tra chính sách nhóm (Group Policy) 🏢

Nếu bạn đang dùng máy tính của công ty, có thể quản trị viên đã vô hiệu hóa `winget` thông qua Group Policy.

1.  Nhấn `Windows` + `R`, gõ `gpedit.msc` và Enter (lưu ý: `gpedit.msc` không có trên phiên bản Windows Home).
2.  Điều hướng đến:
    `Computer Configuration > Administrative Templates > Windows Components > Desktop App Installer`
3.  Ở bên phải, tìm các chính sách có tên như **"Turn on App Installer"** hoặc **"Enable winget"**. Đảm bảo chúng được đặt thành **"Not Configured"** hoặc **"Enabled"**. Nếu chúng đang ở trạng thái **"Disabled"**, đó chính là nguyên nhân. Bạn cần liên hệ với quản trị viên IT để thay đổi.
