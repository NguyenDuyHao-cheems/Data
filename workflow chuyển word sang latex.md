Dưới đây là workflow tái sử dụng để chuyển nội dung báo cáo từ file Word `.docx` sang LaTeX

**Workflow Tổng Quát**
- **Mục tiêu:** Chuyển nội dung Word sang LaTeX, giữ nguyên ý và số liệu, đồng thời cải thiện trình bày.
- **Đầu vào:** File `.docx` chứa báo cáo kỹ thuật, công thức, bảng, ký hiệu toán học.
- **Đầu ra:** File `main.tex` biên dịch được bằng `lualatex`/`xelatex`, có bố cục dễ đọc, công thức đúng chuẩn LaTeX.
- **Nguyên tắc chính:** Không thay đổi nội dung học thuật, chỉ chuẩn hóa định dạng và cú pháp LaTeX.

**1. Chuẩn Bị LaTeX**
- Dùng document class đơn giản:
  - `\documentclass[11pt]{article}`
- Dùng lề tiêu chuẩn:
  - `\usepackage[margin=1in]{geometry}`
- Hỗ trợ tiếng Việt và Unicode:
  - `\usepackage{fontspec}`
  - `\usepackage{unicode-math}`
- Hỗ trợ công thức:
  - `\usepackage{amsmath}`
- Hỗ trợ bảng dài:
  - `\usepackage{longtable}`
  - `\usepackage{array}`
  - `\usepackage{makecell}`

**2. Font Và Trình Biên Dịch**
- Dùng `lualatex` hoặc `xelatex`, không dùng `pdflatex` cho tài liệu tiếng Việt/Unicode.
- Thiết lập font:
  - `\setmainfont{DejaVu Serif}`
  - `\setmathfont{Latin Modern Math}`
- Nếu dùng Prism hoặc editor hỗ trợ magic comment:
  - `% !TEX program = xelatex`

**3. Quy Tắc Giữ Nội Dung**
- Giữ nguyên:
  - Số liệu tính toán.
  - Tên mục.
  - Thứ tự mục.
  - Nội dung diễn giải.
  - Ký hiệu kỹ thuật.
  - Đơn vị.
- Không tự sửa kết quả tính toán nếu người dùng không yêu cầu.
- Nếu công thức Word bị trích xuất thiếu, ưu tiên khôi phục theo ngữ cảnh toán học thay vì bỏ trống.
- Các kết quả tính toán làm tròn 4 chữ số phải được làm tròn lại 2 chữ số và báo cáo lại cho người dùng.

**4. Định Dạng Tiêu Đề**
- Với tiêu đề chương lớn:
  - Dùng căn giữa và in đậm:
    ```latex
    \begin{center}
    \Large\bfseries CHƯƠNG III: ...
    \end{center}
    ```
- Với mục chính/phụ:
  - Dùng:
    ```latex
    \textbf{3.3.1. Khoảng cách trục sơ bộ}
    ```
- Sau mỗi tiêu đề nên có một dòng trống để dễ đọc trong source và PDF.

**5. Định Dạng Đoạn Văn**
- Dùng:
  ```latex
  \setlength{\parindent}{0pt}
  \setlength{\parskip}{0.75\baselineskip}
  ```
- Tách các dòng nội dung dài thành các đoạn ngắn.
- Mỗi ý tính toán hoặc kết luận nên nằm trên dòng riêng.
- Không nhồi nhiều câu/công thức vào cùng một đoạn nếu làm PDF khó đọc.

**6. Quy Tắc Công Thức Toán**
- Mọi ký hiệu toán học phải đặt trong `$...$` hoặc `\[...\]`.
- Công thức ngắn đặt inline:
  ```latex
  $L_h = 8 \times 2 \times 300 \times 5 = 24000$
  ```
- Công thức dài đặt display:
  ```latex
  \[
  N_{HE1}=60c\sum_{i=1}^{n}
  \left(\frac{T_i}{T_{\max}}\right)^3 n_i t_i
  \]
  ```
- Không để ký hiệu toán dạng chữ thường như:
  ```latex
  Lh = 8 × 2
  ```
- Nên đổi thành:
  ```latex
  $L_h = 8 \times 2$
  ```

**7. Chuẩn Hóa Ký Hiệu**
- Chỉ số dưới:
  - `T1` → `$T_1$`
  - `z2` → `$z_2$`
  - `KHL1` → `$K_{HL1}$`
- Chỉ số trên:
  - `10^7` → `$10^7$`
  - `HB2,4` → `$HB^{2,4}$`
- Ký hiệu Hy Lạp:
  - `σ` → `\sigma`
  - `β` → `\beta`
  - `α` → `\alpha`
  - `ψ` → `\psi`
  - `ε` → `\varepsilon`
- Dấu toán học:
  - `×` → `\times`
  - `÷` → `\div`
  - `≤` → `\le`
  - `≥` → `\ge`
  - `=>` → `\Rightarrow`
  - `→` → `\rightarrow`

**8. Chuẩn Hóa Đơn Vị**
- Đơn vị trong công thức nên dùng `\text{...}`:
  ```latex
  $445,91\ \text{MPa}$
  ```
- Ví dụ:
  ```latex
  $T_1 = 83715,17\ \text{Nmm}$
  ```
  ```latex
  $v = 0,55\ \text{m/s}$
  ```
  ```latex
  $K_a = 49,5\ \text{MPa}^{1/3}$
  ```
- Không nên viết đơn vị trần trong math mode:
  ```latex
  $445,91 MPa$
  ```
- Nên viết:
  ```latex
  $445,91\ \text{MPa}$
  ```

**9. Công Thức Dài**
- Công thức quá dài nên dùng `\[...\]`.
- Nếu vẫn bị tràn dòng, có thể dùng:
  ```latex
  \begin{aligned}
  ...
  \end{aligned}
  ```
- Ví dụ:
  ```latex
  \[
  \begin{aligned}
  N_{HE1}
  &=60c\sum_{i=1}^{n}
  \left(\frac{T_i}{T_{\max}}\right)^3 n_i t_i \\
  &=5,05\times10^8\ \text{(chu kỳ)}
  \end{aligned}
  \]
  ```

**10. Quy Tắc Bảng**
- Với bảng dài hoặc nhiều dòng, dùng `longtable`.
- Không dùng cột quá hẹp cho công thức.
- Nên định nghĩa độ rộng cột bằng `p{...}`:
  ```latex
  \begin{longtable}{|p{0.24\textwidth}|p{0.10\textwidth}|p{0.28\textwidth}|p{0.28\textwidth}|}
  ```
- Tăng chiều cao hàng:
  ```latex
  \renewcommand{\arraystretch}{1.45}
  ```
- Giảm khoảng đệm ngang nếu bảng rộng:
  ```latex
  \setlength{\tabcolsep}{4pt}
  ```
- Dùng `\multicolumn` cho các hàng chỉ có một giá trị:
  ```latex
  Khoảng cách trục & $a$ & \multicolumn{2}{l|}{$160\ \text{mm}$} \\ \hline
  ```

**11. Chống Chồng Chữ Trong Bảng**
- Không nhét công thức dài vào cột quá nhỏ.
- Tách công thức cho bánh 1 và bánh 2 thành hai cột riêng.
- Dùng font nhỏ cho bảng:
  ```latex
  {\small
  ...
  }
  ```
- Tránh `\makecell` nếu nội dung là công thức dài; ưu tiên cột `p{...}` và math inline gọn.

**12. Checklist Sau Khi Chuyển Đổi**
- Kiểm tra tiêu đề:
  - Có xuống dòng sau tiêu đề không?
  - Mục chính/phụ có rõ không?
- Kiểm tra công thức:
  - Mọi ký hiệu toán đã nằm trong `$...$` hoặc `\[...\]` chưa?
  - Có còn `×`, `≤`, `≥`, `=>` ngoài math mode không?
- Kiểm tra bảng:
  - Có bị đè chữ không?
  - Cột có đủ rộng không?
  - Công thức có bị tràn quá nhiều không?
- Kiểm tra biên dịch:
  ```bash
  lualatex -interaction=nonstopmode -halt-on-error main.tex
  ```
- Kiểm tra PDF bằng render ảnh:
  ```bash
  gs -q -dSAFER -dBATCH -dNOPAUSE -sDEVICE=png16m -r120 -dFirstPage=1 -dLastPage=1 -sOutputFile=preview/page-01.png main.pdf
  ```

**13. Quy Trình Thực Thi Khuyến Nghị**
1. Trích xuất văn bản từ `.docx`.
2. Đưa toàn bộ nội dung vào `main.tex`.
3. Thiết lập preamble hỗ trợ tiếng Việt, toán, bảng.
4. Chia lại tiêu đề và đoạn văn.
5. Chuyển ký hiệu toán sang `$...$`.
6. Chuyển công thức dài sang `\[...\]`.
7. Chuẩn hóa bảng bằng `longtable`.
8. Biên dịch bằng `lualatex`.
9. Render PDF thành ảnh để kiểm tra trực quan.
10. Sửa lỗi chồng chữ, tràn dòng, thiếu math mode.
11. Biên dịch lại lần cuối.

**14. Template Rút Gọn**
```latex
% !TEX program = xelatex
\documentclass[11pt]{article}
\usepackage[margin=1in]{geometry}
\usepackage{fontspec}
\usepackage{amsmath}
\usepackage{unicode-math}
\usepackage{longtable}
\usepackage{array}
\usepackage{makecell}

\setmainfont{DejaVu Serif}
\setmathfont{Latin Modern Math}
\setlength{\parindent}{0pt}
\setlength{\parskip}{0.75\baselineskip}

\begin{document}

\begin{center}
\Large\bfseries TÊN CHƯƠNG
\end{center}

\textbf{1.1. Tên mục}

Nội dung văn bản.

Công thức ngắn: $L_h = 8 \times 2 \times 300 \times 5 = 24000$.

Công thức dài:
\[
N_{HE1}=60c\sum_{i=1}^{n}
\left(\frac{T_i}{T_{\max}}\right)^3 n_i t_i
\]

{\small
\renewcommand{\arraystretch}{1.45}
\setlength{\tabcolsep}{4pt}
\begin{longtable}{|p{0.24\textwidth}|p{0.10\textwidth}|p{0.28\textwidth}|p{0.28\textwidth}|}
\hline
\textbf{Thông số} & \textbf{Ký hiệu} & \multicolumn{2}{c|}{\textbf{Công thức tính}} \\ \hline
Khoảng cách trục & $a$ & \multicolumn{2}{l|}{$160\ \text{mm}$} \\ \hline
Số răng & $z$ & $z_1=34$ răng & $z_2=136$ răng \\ \hline
\end{longtable}
}

\end{document}
```