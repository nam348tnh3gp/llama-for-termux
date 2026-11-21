# llama-for-termux

[English](#english) | [Tiếng Việt](#tiếng-việt)

---

## English

Prebuilt [llama.cpp](https://github.com/ggml-org/llama.cpp) for **Termux (Android)**, built with GitHub Actions and packaged as `.deb` files so you can install it with one command instead of compiling on your phone.

### Supported architectures

| Termux arch | Android ABI | Status |
| --- | --- | --- |
| `aarch64` | `arm64-v8a` | main (KleidiAI enabled) |
| `x86_64` | `x86_64` | main (baseline SSE; optional AVX2 build) |
| `arm` | `armeabi-v7a` | experimental |
| `i686` | `x86` | experimental |

Find your architecture with `dpkg --print-architecture` in Termux. Experimental builds may fail to compile with newer llama.cpp commits. If one fails, the release is still published without it, so the package for your architecture may be missing from some releases.

### Install

Requirements: Termux (F-Droid or GitHub build), Android 9 or newer.

```bash
pkg update
pkg install curl

# Pick the package for your architecture
ARCH=$(dpkg --print-architecture)
API=https://api.github.com/repos/nam348tnh3gp/llama-for-termux/releases/latest
curl -fsSL "$API" | grep -o "https://[^\"]*\(_${ARCH}\.deb\|SHA256SUMS\)" | xargs -n1 curl -fLO

# Verify, then install
sha256sum -c --ignore-missing SHA256SUMS
pkg install ./llama-termux_*_${ARCH}.deb
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
- On 32-bit builds (`arm`, `i686`) a process can address only a few GB, so use small models.

### Build configuration

| Item | Value |
| --- | --- |
| Source | `ggml-org/llama.cpp` (branch `master` by default, selectable when running the workflow; all architectures in one run use the same commit) |
| Toolchain | Android NDK r27d, `android-28` |
| Build type | Release, static, LTO, ccache |
| CPU | Baseline instruction sets, so it runs on any device of that architecture. Optional inputs enable more (see below) |
| Optimizations | KleidiAI on `aarch64` only |
| Disabled | OpenMP, llamafile, libcurl, OpenSSL, tests, examples |
| Output | `llama-termux_<version>_<arch>.deb` + one shared `SHA256SUMS` |

Package versions look like `0.<date>.<run number>.<llama.cpp commit>`. Each release page states the llama.cpp commit that was used.

### Build it yourself with GitHub Actions

1. Fork this repo.
2. Go to **Actions → Build llama.cpp for Android (Termux) → Run workflow**.
3. Inputs:
   - `llama_ref`: llama.cpp branch, tag or commit (pin a tag or commit for reproducible builds).
   - `archs`: comma-separated list, default `aarch64,arm,i686,x86_64`.
   - `tag_name`, `release_name`: leave empty to auto-generate.
   - `make_latest`, `prerelease`: release flags.
   - `cpu_arch` (`aarch64` only): for example `armv8.2-a+dotprod`. Use only if you are sure your device supports it.
   - `x86_avx2` (`x86_64` only): enables AVX2/FMA/F16C. Use only if the CPU supports them.
4. The workflow builds each architecture in parallel, then creates one tag and one release with all `.deb` files and `SHA256SUMS`.

The workflow fails if a main architecture (`aarch64`, `x86_64`) fails, if `llama-cli` or `llama-server` is missing, or if a binary does not match the target architecture.

### Troubleshooting

- **`Illegal instruction`**: you used a build with `cpu_arch` or `x86_avx2` that your CPU does not support. Use the baseline build.
- **Killed / Termux closes suddenly**: out of memory. Lower the context (`-c`), use a smaller model or a lower-bit quantization.
- **No package for my architecture**: it is probably `arm` or `i686` (experimental) and failed to build in that release. Try an older release or build it yourself.
- **Conflict with Termux's `llama-cpp` package**: this package declares `Conflicts: llama-cpp`, so remove the other one first.
- **Termux from Google Play**: that build is outdated and often broken. Use F-Droid or GitHub.
- **GPU (Vulkan/OpenCL)**: not supported in this build.

### License

llama.cpp is released under the MIT license. The binaries in this package are built from that project's source. See [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) for details.

---

## Tiếng Việt

Build sẵn [llama.cpp](https://github.com/ggml-org/llama.cpp) cho **Termux (Android)** bằng GitHub Actions, đóng gói thành các file `.deb` để cài một lệnh, không cần tự biên dịch trên điện thoại.

### Kiến trúc được hỗ trợ

| Kiến trúc Termux | Android ABI | Trạng thái |
| --- | --- | --- |
| `aarch64` | `arm64-v8a` | chính (bật KleidiAI) |
| `x86_64` | `x86_64` | chính (baseline SSE; có tùy chọn build AVX2) |
| `arm` | `armeabi-v7a` | thử nghiệm |
| `i686` | `x86` | thử nghiệm |

Xem kiến trúc của máy bằng `dpkg --print-architecture` trong Termux. Các bản thử nghiệm có thể không build được với commit llama.cpp mới. Nếu một bản lỗi, release vẫn được phát hành nhưng thiếu bản đó, nên có release sẽ không có gói cho kiến trúc của bạn.

### Cài đặt

Yêu cầu: Termux (bản F-Droid hoặc GitHub), Android 9 trở lên.

```bash
pkg update
pkg install curl

# Chọn gói đúng kiến trúc
ARCH=$(dpkg --print-architecture)
API=https://api.github.com/repos/nam348tnh3gp/llama-for-termux/releases/latest
curl -fsSL "$API" | grep -o "https://[^\"]*\(_${ARCH}\.deb\|SHA256SUMS\)" | xargs -n1 curl -fLO

# Kiểm tra toàn vẹn rồi cài
sha256sum -c --ignore-missing SHA256SUMS
pkg install ./llama-termux_*_${ARCH}.deb
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
- Với bản 32-bit (`arm`, `i686`), mỗi tiến trình chỉ địa chỉ hóa được vài GB, nên hãy dùng model nhỏ.

### Cấu hình build

| Mục | Giá trị |
| --- | --- |
| Nguồn | `ggml-org/llama.cpp` (mặc định nhánh `master`, đổi được khi chạy workflow; mọi kiến trúc trong cùng một lần chạy dùng chung một commit) |
| Toolchain | Android NDK r27d, `android-28` |
| Kiểu build | Release, build tĩnh, LTO, ccache |
| CPU | Tập lệnh baseline nên chạy được trên mọi máy cùng kiến trúc. Có tùy chọn bật thêm (xem bên dưới) |
| Tối ưu | KleidiAI, chỉ trên `aarch64` |
| Tắt | OpenMP, llamafile, libcurl, OpenSSL, tests, examples |
| Đầu ra | `llama-termux_<version>_<arch>.deb` + một file `SHA256SUMS` chung |

Phiên bản gói có dạng `0.<ngày>.<số run>.<commit llama.cpp>`. Trang release ghi rõ commit llama.cpp đã dùng.

### Tự build bằng GitHub Actions

1. Fork repo này.
2. Vào **Actions → Build llama.cpp for Android (Termux) → Run workflow**.
3. Các tham số:
   - `llama_ref`: nhánh, tag hoặc commit của llama.cpp (nên ghim tag hoặc commit để tái lập được).
   - `archs`: danh sách cách nhau bằng dấu phẩy, mặc định `aarch64,arm,i686,x86_64`.
   - `tag_name`, `release_name`: để trống sẽ tự sinh.
   - `make_latest`, `prerelease`: đánh dấu release.
   - `cpu_arch` (chỉ `aarch64`): ví dụ `armv8.2-a+dotprod`. Chỉ dùng nếu chắc chắn máy hỗ trợ.
   - `x86_avx2` (chỉ `x86_64`): bật AVX2/FMA/F16C. Chỉ dùng nếu CPU hỗ trợ.
4. Workflow build song song từng kiến trúc, rồi tạo một tag và một release chứa tất cả file `.deb` cùng `SHA256SUMS`.

Workflow sẽ fail nếu một kiến trúc chính (`aarch64`, `x86_64`) lỗi, nếu thiếu `llama-cli` hoặc `llama-server`, hoặc nếu binary không khớp kiến trúc đích.

### Xử lý sự cố

- **`Illegal instruction`**: bạn đã dùng bản build có `cpu_arch` hoặc `x86_avx2` mà CPU không hỗ trợ. Dùng bản baseline.
- **Bị đóng đột ngột / `Killed`**: thiếu RAM. Giảm context (`-c`), dùng model nhỏ hơn hoặc lượng tử hóa thấp hơn.
- **Không có gói cho kiến trúc của mình**: nhiều khả năng là `arm` hoặc `i686` (thử nghiệm) và bản build trong release đó đã lỗi. Thử release cũ hơn hoặc tự build.
- **Xung đột với gói `llama-cpp` của Termux**: gói này khai báo `Conflicts: llama-cpp`, nên cần gỡ bản kia trước.
- **Termux từ Google Play**: bản này cũ, dễ lỗi. Nên dùng F-Droid hoặc GitHub.
- **Cần dùng GPU (Vulkan/OpenCL)**: chưa hỗ trợ trong bản build này.

### Giấy phép

llama.cpp dùng giấy phép MIT. Các binary trong gói được build từ mã nguồn gốc của dự án đó. Xem kho [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) để biết chi tiết.
