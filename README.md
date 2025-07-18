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
