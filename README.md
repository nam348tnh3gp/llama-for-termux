# llama-for-termux

[English](#english) | [Tiếng Việt](#tiếng-việt)

---

## English

Prebuilt [llama.cpp](https://github.com/ggml-org/llama.cpp) for **Termux (Android arm64)**, built with GitHub Actions and packaged as a `.deb` so you can install it with one command instead of compiling on your phone.

### Install

Requirements: Termux (F-Droid or GitHub build), Android 9 or newer, arm64.

```bash
pkg update
pkg install curl

# Download the latest release and its checksum file
API=https://api.github.com/repos/nam348tnh3gp/llama-for-termux/releases/latest
curl -fsSL "$API" | grep -o 'https://[^"]*\(\.deb\|SHA256SUMS\)' | xargs -n1 curl -fLO

# Verify, then install
sha256sum -c SHA256SUMS
pkg install ./llama-termux_*.deb
```

Or download manually from the [Releases](https://github.com/nam348tnh3gp/llama-for-termux/releases) page.

Uninstall: `pkg uninstall llama-termux`

### Usage

The package installs `llama-cli`, `llama-server`, `llama-bench`, `llama-quantize` (when present in the build) and `llama-check-ram`.

This build is compiled **without libcurl**, so the `-hf` option (download from Hugging Face) does not work. Download a GGUF file yourself:

```bash
curl -L -o ~/model.gguf "<URL of a .gguf file on Hugging Face>"
```

Chat in the terminal:

```bash
llama-cli -m ~/model.gguf -c 4096
```

Run an OpenAI-compatible server:

```bash
llama-server -m ~/model.gguf -c 4096 --host 127.0.0.1 --port 8080
```

Tips:
- Keep models in `~/` (not `/sdcard`) for faster loading.
- Start with a moderate context size (2048 to 4096). A large context can make Android kill Termux when memory runs out.
- Check free RAM before loading a model: `llama-check-ram 3000` (threshold in MB, default 2048).
- For long runs in the background: `termux-wake-lock`.
- On low-RAM devices, use small models and low-bit quantization (for example 1B to 3B, Q4).

### Build configuration

| Item | Value |
| --- | --- |
| Source | `ggml-org/llama.cpp` (branch `master` by default, selectable when running the workflow) |
| Toolchain | Android NDK r27d, `arm64-v8a`, `android-28` |
| Build type | Release, static, LTO, ccache |
| CPU | Baseline armv8-a (runs on any arm64 device). Optional `cpu_arch` input enables extra instruction sets |
| Optimizations | KleidiAI enabled |
| Disabled | OpenMP, llamafile, libcurl, OpenSSL, tests, examples |
| Output | `llama-termux_<version>_aarch64.deb` + `SHA256SUMS` |

Package versions look like `0.<date>.<run number>.<llama.cpp commit>`. Each release page states the llama.cpp commit that was used.

### Build it yourself with GitHub Actions

1. Fork this repo.
2. Go to **Actions → Build llama.cpp for Android (Termux) → Run workflow**.
3. Inputs:
   - `llama_ref`: llama.cpp branch, tag or commit (pin a tag or commit for reproducible builds).
   - `tag_name`, `release_name`: leave empty to auto-generate.
   - `make_latest`, `prerelease`: release flags.
   - `cpu_arch`: for example `armv8.2-a+dotprod`. Use only if you are sure your device supports it.
4. The workflow creates the tag and a release with the `.deb` attached.

The workflow fails if `llama-cli` or `llama-server` is missing, or if a binary is not an aarch64 ELF.

### Troubleshooting

- **`Illegal instruction`**: you used a build with a `cpu_arch` your CPU does not support. Use the baseline build (leave `cpu_arch` empty).
- **Killed / Termux closes suddenly**: out of memory. Lower the context (`-c`), use a smaller model or a lower-bit quantization.
- **Conflict with Termux's `llama-cpp` package**: this package declares `Conflicts: llama-cpp`, so remove the other one first.
- **Termux from Google Play**: that build is outdated and often broken. Use F-Droid or GitHub.
- **GPU (Vulkan/OpenCL)**: not supported in this build.

### License

llama.cpp is released under the MIT license. The binaries in this package are built from that project's source. See [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) for details.

---

## Tiếng Việt

Build sẵn [llama.cpp](https://github.com/ggml-org/llama.cpp) cho **Termux (Android arm64)** bằng GitHub Actions, đóng gói thành file `.deb` để cài một lệnh, không cần tự biên dịch trên điện thoại.

### Cài đặt

Yêu cầu: Termux (bản F-Droid hoặc GitHub), Android 9 trở lên, kiến trúc arm64.

```bash
pkg update
pkg install curl

# Tải bản release mới nhất và file checksum
API=https://api.github.com/repos/nam348tnh3gp/llama-for-termux/releases/latest
curl -fsSL "$API" | grep -o 'https://[^"]*\(\.deb\|SHA256SUMS\)' | xargs -n1 curl -fLO

# Kiểm tra toàn vẹn rồi cài
sha256sum -c SHA256SUMS
pkg install ./llama-termux_*.deb
```

Hoặc tải thủ công trong trang [Releases](https://github.com/nam348tnh3gp/llama-for-termux/releases).

Gỡ cài đặt: `pkg uninstall llama-termux`

### Sử dụng

Gói cài các lệnh `llama-cli`, `llama-server`, `llama-bench`, `llama-quantize` (nếu có trong bản build) và `llama-check-ram`.

Bản build **tắt libcurl**, nên không dùng được tùy chọn tải model qua `-hf`. Hãy tự tải file GGUF:

```bash
curl -L -o ~/model.gguf "<URL file .gguf trên Hugging Face>"
```

Chat trong terminal:

```bash
llama-cli -m ~/model.gguf -c 4096
```

Chạy server tương thích OpenAI API:

```bash
llama-server -m ~/model.gguf -c 4096 --host 127.0.0.1 --port 8080
```

Gợi ý:
- Đặt model trong `~/` (không phải `/sdcard`) để đọc nhanh hơn.
- Bắt đầu với context vừa phải (2048 đến 4096). Context lớn dễ làm Android đóng Termux vì thiếu RAM.
- Kiểm tra RAM trống trước khi nạp model: `llama-check-ram 3000` (ngưỡng MB, mặc định 2048).
- Chạy lâu ở nền: `termux-wake-lock`.
- Chọn model nhỏ và lượng tử hóa thấp (ví dụ 1B đến 3B, Q4) nếu máy ít RAM.

### Cấu hình build

| Mục | Giá trị |
| --- | --- |
| Nguồn | `ggml-org/llama.cpp` (mặc định nhánh `master`, đổi được khi chạy workflow) |
| Toolchain | Android NDK r27d, `arm64-v8a`, `android-28` |
| Kiểu build | Release, build tĩnh, LTO, ccache |
| CPU | Baseline armv8-a (chạy được trên mọi máy arm64). Tùy chọn `cpu_arch` để bật thêm tập lệnh |
| Tối ưu | KleidiAI bật |
| Tắt | OpenMP, llamafile, libcurl, OpenSSL, tests, examples |
| Đầu ra | `llama-termux_<version>_aarch64.deb` + `SHA256SUMS` |

Phiên bản gói có dạng `0.<ngày>.<số run>.<commit llama.cpp>`. Trang release ghi rõ commit llama.cpp đã dùng.

### Tự build bằng GitHub Actions

1. Fork repo này.
2. Vào **Actions → Build llama.cpp for Android (Termux) → Run workflow**.
3. Các tham số:
   - `llama_ref`: nhánh, tag hoặc commit của llama.cpp (nên ghim tag hoặc commit để tái lập được).
   - `tag_name`, `release_name`: để trống sẽ tự sinh.
   - `make_latest`, `prerelease`: đánh dấu release.
   - `cpu_arch`: ví dụ `armv8.2-a+dotprod`. Chỉ dùng nếu chắc chắn máy hỗ trợ.
4. Workflow tự tạo tag và release kèm file `.deb`.

Workflow sẽ fail nếu thiếu `llama-cli` hoặc `llama-server`, hoặc nếu binary không phải ELF aarch64.

### Xử lý sự cố

- **`Illegal instruction`**: bạn đã dùng bản build với `cpu_arch` mà CPU không hỗ trợ. Dùng bản baseline (để trống `cpu_arch`).
- **Bị đóng đột ngột / `Killed`**: thiếu RAM. Giảm context (`-c`), dùng model nhỏ hơn hoặc lượng tử hóa thấp hơn.
- **Xung đột với gói `llama-cpp` của Termux**: gói này khai báo `Conflicts: llama-cpp`, nên cần gỡ bản kia trước.
- **Termux từ Google Play**: bản này cũ, dễ lỗi. Nên dùng F-Droid hoặc GitHub.
- **Cần dùng GPU (Vulkan/OpenCL)**: chưa hỗ trợ trong bản build này.

### Giấy phép

llama.cpp dùng giấy phép MIT. Các binary trong gói được build từ mã nguồn gốc của dự án đó. Xem kho [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) để biết chi tiết.
