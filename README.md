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
    Tự động cài đặt một danh sách các phần mềm phát triển và công cụ bằng Winget.

.DESCRIPTION
    Tập lệnh này được thiết kế để tự động hóa việc thiết lập một môi trường phát triển mới.
    Nó cài đặt các ứng dụng, công cụ dòng lệnh, SDK và các phần mềm khác được liệt kê.
    
    LƯU Ý: Chạy tập lệnh này với quyền Quản trị viên (Run as Administrator).
    Tùy chỉnh các phần mềm được cài đặt bằng cách thêm/xóa dấu thăng (#) ở đầu mỗi dòng.

.AUTHOR
    Dựa trên yêu cầu của người dùng.

.VERSION
    1.0
#>

#================================================================================
# HÀM TRỢ GIÚP
#================================================================================

# Hàm để cài đặt phần mềm thương mại hoặc phần mềm yêu cầu tệp cài đặt cục bộ
Function Install-LocalOrCommercial-Software {
    param(
        [string]$SoftwareName,
        [string]$InstallerName,
        [string]$SilentArgs
    )

    $installerPath = Join-Path $PSScriptRoot $InstallerName
    
    if (Test-Path $installerPath) {
        Write-Host "Bắt đầu cài đặt $SoftwareName..." -ForegroundColor Green
        try {
            Start-Process -FilePath $installerPath -ArgumentList $SilentArgs -Wait -PassThru -ErrorAction Stop
            Write-Host "$SoftwareName đã được cài đặt thành công." -ForegroundColor Green
        }
        catch {
            Write-Error "Lỗi khi cài đặt $SoftwareName. Chi tiết: $_"
        }
    }
    else {
        Write-Warning "Không tìm thấy tệp cài đặt cho $SoftwareName tại '$installerPath'. Vui lòng tải về và đặt vào cùng thư mục với tập lệnh."
    }
}

# Hàm để cài đặt Visual Studio 2022 Community với các workload được chỉ định
Function Install-VisualStudio {
    Write-Host "Bắt đầu cài đặt Visual Studio 2022 Community..." -ForegroundColor Cyan
    try {
        $vsInstallerUrl = "https://aka.ms/vs/17/release/vs_Community.exe"
        $vsInstallerPath = Join-Path $env:TEMP "vs_Community.exe"
        
        Invoke-WebRequest -Uri $vsInstallerUrl -OutFile $vsInstallerPath
        
        # Thêm các workload cần thiết vào đây. Ví dụ: Phát triển .NET, Desktop C++, Web, Azure.
        # Để xem danh sách đầy đủ các ID workload, hãy truy cập: https://learn.microsoft.com/en-us/visualstudio/install/workload-and-component-ids
        $workloads = @(
            "--add Microsoft.VisualStudio.Workload.ManagedDesktop", # .NET desktop development
            "--add Microsoft.VisualStudio.Workload.NativeDesktop",   # Desktop development with C++
            "--add Microsoft.VisualStudio.Workload.NetWeb",        # ASP.NET and web development
            "--add Microsoft.VisualStudio.Workload.Azure",         # Azure development
            "--add Microsoft.VisualStudio.Workload.Data",          # Data storage and processing
            "--includeRecommended"
        )
        
        $arguments = "$workloads --quiet --norestart"
        
        Start-Process -FilePath $vsInstallerPath -ArgumentList $arguments -Wait -PassThru
        Write-Host "Visual Studio 2022 Community đã được cài đặt." -ForegroundColor Green
    }
    catch {
        Write-Error "Lỗi khi cài đặt Visual Studio. Chi tiết: $_"
    }
}

#================================================================================
# CÀI ĐẶT PHẦN MỀM
#================================================================================

Write-Host "Bắt đầu quá trình cài đặt phần mềm. Vui lòng đảm bảo bạn đang chạy với quyền Quản trị viên." -ForegroundColor Yellow
Write-Host "----------------------------------------------------------------"

# --- MÔI TRƯỜNG PHÁT TRIỂN & IDEs ---
Write-Host "Đang cài đặt Môi trường phát triển & IDEs..." -ForegroundColor Cyan
Install-VisualStudio
winget install --id Microsoft.VisualStudioCode -e --accept-package-agreements # Visual Studio Code
winget install --id Oracle.Java.JDK.18 -e --accept-package-agreements         # Java SE Development Kit 18
winget install --id Oracle.Java.JDK.8 -e --accept-package-agreements          # Java SE Development Kit 8
winget install --id Python.Python.3.10 -e --accept-package-agreements         # Python 3.10
winget install --id Gluon.SceneBuilder -e --accept-package-agreements         # SceneBuilder
winget install --id Node.js -e --accept-package-agreements                    # Node.js
# winget install --id Microsoft.VisualStudio.2019.Tools.Unity -e --accept-package-agreements # VS 2019 Tools for Unity (Nếu cần)


# --- CƠ SỞ DỮ LIỆU ---
Write-Host "Đang cài đặt Công cụ Cơ sở dữ liệu..." -ForegroundColor Cyan
winget install --id Oracle.MySQL.Server -e --accept-package-agreements --scope machine # MySQL Server
winget install --id Oracle.MySQL.Workbench -e --accept-package-agreements             # MySQL Workbench
winget install --id Oracle.MySQL.Shell -e --accept-package-agreements                 # MySQL Shell
winget install --id Oracle.MySQL.Connector.ODBC -e --accept-package-agreements        # MySQL Connector/ODBC
winget install --id Oracle.MySQL.Connector.CPP -e --accept-package-agreements         # MySQL Connector C++
winget install --id Oracle.MySQL.Connector.NET -e --accept-package-agreements         # MySQL Connector Net
winget install --id Microsoft.SQLServer.2019.Developer -e --accept-package-agreements # SQL Server 2019 Developer Edition
winget install --id Microsoft.SQLServerManagementStudio -e --accept-package-agreements    # SQL Server Management Studio (SSMS)


# --- CÔNG CỤ & TIỆN ÍCH ---
Write-Host "Đang cài đặt Công cụ & Tiện ích..." -ForegroundColor Cyan
winget install --id 7zip.7zip -e --accept-package-agreements                   # 7-Zip
winget install --id Git.Git -e --accept-package-agreements                     # Git
winget install --id Apache.OpenOffice -e --accept-package-agreements           # OpenOffice
winget install --id DevinCook.Flowgorithm -e --accept-package-agreements       # Flowgorithm
winget install --id Microsoft.Edge -e --accept-package-agreements              # Microsoft Edge (Thường đã được cài sẵn)
# winget install --id Microsoft.Silverlight -e --accept-package-agreements     # Microsoft Silverlight (Lỗi thời, không khuyến khích)


# --- PHẦN MỀM THƯƠNG MẠI & YÊU CẦU TỆP CÀI ĐẶT CỤC BỘ ---
# Hướng dẫn: Tải tệp cài đặt (.exe hoặc .msi) từ trang web chính thức
# và đặt nó vào cùng thư mục với tập lệnh này. Sau đó bỏ ghi chú dòng tương ứng.

Write-Host "Đang xử lý phần mềm thương mại (yêu cầu tệp cài đặt cục bộ)..." -ForegroundColor Cyan

# Ví dụ cho VMware Workstation
# Tải tệp cài đặt ví dụ: VMware-workstation-full-xx.x.x-xxxx.exe
# Tham số im lặng có thể khác nhau, kiểm tra tài liệu của nhà cung cấp.
# Install-LocalOrCommercial-Software -SoftwareName "VMware Workstation" -InstallerName "VMware-workstation-full-xx.x.x-xxxx.exe" -SilentArgs "/S /v /qn"

# Ví dụ cho MYOB AccountRight
# Tải về và điền tên tệp cài đặt chính xác.
# Install-LocalOrCommercial-Software -SoftwareName "MYOB AccountRight" -InstallerName "MYOB_AccountRight_2018.3.exe" -SilentArgs "/S"

# Ví dụ cho NetSupport School
# Install-LocalOrCommercial-Software -SoftwareName "NetSupport School" -InstallerName "NetSupport_School_Tutor.exe" -SilentArgs "/s /v/qn"


# --- DRIVERS & PHẦN MỀM OEM ---
# Việc cài đặt drivers thường được xử lý tốt nhất bởi Windows Update hoặc các công cụ hỗ trợ của nhà sản xuất (ví dụ: Dell SupportAssist).
# Tuy nhiên, bạn có thể thử cài đặt chúng qua winget.
Write-Host "Đang cài đặt Drivers & Phần mềm OEM..." -ForegroundColor Cyan
winget install --id Dell.SupportAssist -e --accept-package-agreements         # Dell SupportAssist
winget install --id Intel.WirelessBluetooth -e --accept-package-agreements    # Intel Wireless Bluetooth
winget install --id Intel.ChipsetDeviceSoftware -e --accept-package-agreements# Intel Chipset Software

Write-Host "----------------------------------------------------------------"
Write-Host "Tất cả các tác vụ đã hoàn tất!" -ForegroundColor Green

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
