# Folder Structure Specification
# Đặc tả Cấu trúc Thư mục

## 1. Principles / Nguyên tắc
The curriculum must be highly organized, deterministic, and easy to navigate programmatically or manually.
Giáo trình phải được tổ chức cao, mang tính xác định và dễ dàng điều hướng bằng chương trình hoặc thủ công.

## 2. Root Structure / Cấu trúc Gốc
```text
Keycloak-Enterprise-Curriculum/
├── Software-Specification/    # Rules and conventions
├── 01-Introduction/           # Chapter 1
├── 02-IAM/                    # Chapter 2
...
└── 50-Capstone-Project/       # Chapter 50
```

## 3. Chapter Structure / Cấu trúc cấp Chương
Each chapter directory MUST contain the following structure:
Mỗi thư mục chương PHẢI chứa cấu trúc sau:

```text
XX-Chapter-Name/
├── README.md                      # Chapter overview and objectives
├── Module-1-Name/                 # Logical grouping of lessons
│   ├── Lesson-1-Name.md           # The actual learning content
│   ├── Lesson-2-Name.md
│   ├── diagrams/                  # Raw mermaid files or images (if any)
│   ├── assets/                    # Screenshots, logos
│   └── code/                      # Runnable source code
│       ├── spring-boot-app/       # Full application source
│       ├── docker-compose.yml     # Infrastructure setup
│       └── README.md              # Instructions to run the code
├── Module-2-Name/
└── Labs/                          # Hands-on exercises
    ├── Lab-1.md
    └── solution/
```

## 4. Rule Enforcement / Thực thi Quy tắc
- AI must strictly follow this structure when generating any content.
- AI phải tuân thủ nghiêm ngặt cấu trúc này khi sinh bất kỳ nội dung nào.
- Empty folders should be avoided unless necessary for future-proofing.
- Nên tránh các thư mục trống trừ khi cần thiết cho việc mở rộng trong tương lai.
