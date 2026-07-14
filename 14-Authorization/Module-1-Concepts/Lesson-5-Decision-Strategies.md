> [!NOTE]
> **Category:** Theory (Lý thuyết)
> **Goal:** Hiểu sâu về các Chiến lược Ra quyết định (Decision Strategies) trong Keycloak Authorization Services và cách áp dụng chúng để giải quyết các bài toán phân quyền phức tạp.

## 1. Lý thuyết chuyên sâu (Detailed Theory)

Trong **Keycloak Authorization Services (UMA 2.0 / OAuth2)**, khi một tài nguyên (Resource) hoặc một phạm vi (Scope) được bảo vệ bởi nhiều chính sách (Policies) hoặc quyền (Permissions) khác nhau, hệ thống cần một cách để tổng hợp kết quả của tất cả các chính sách đó và đưa ra quyết định cuối cùng (Cấp quyền - GRANT hoặc Từ chối - DENY). Cơ chế này được gọi là **Decision Strategy**.

Decision Strategy quyết định tính chất khắt khe hay nới lỏng của quy trình cấp quyền. 

Keycloak hỗ trợ 3 chiến lược chính:
1. **AFFIRMATIVE (Khẳng định):** Chỉ cần **MỘT** policy đánh giá thành công (GRANT), quyền truy cập sẽ được cấp. Các policy khác bị từ chối cũng không sao. Đây là chiến lược khoan dung nhất (Logic OR).
2. **UNANIMOUS (Đồng thuận tuyệt đối):** **TẤT CẢ** các policy liên kết phải đánh giá thành công (GRANT). Nếu chỉ cần một policy thất bại (DENY), quyền truy cập lập tức bị từ chối. Đây là chiến lược khắt khe nhất (Logic AND).
3. **CONSENSUS (Đa số tán thành):** Số lượng policy trả về kết quả GRANT phải **lớn hơn** số lượng trả về DENY. Nếu số GRANT bằng số DENY, hoặc tất cả các policy đều bỏ phiếu trắng, Keycloak sẽ đánh giá theo một hành vi fallback được cấu hình mặc định (thường là DENY để đảm bảo an toàn).

**Tại sao cần Decision Strategy?**
Trong các hệ thống phân tán, quyền không chỉ dựa trên Role. Có thể bạn có: `Policy A (Chỉ Role Manager)` và `Policy B (Chỉ làm việc trong giờ hành chính)`. Nếu áp dụng Affirmative, Manager có thể truy cập bất kỳ lúc nào. Nếu áp dụng Unanimous, Manager chỉ được truy cập vào giờ hành chính. Decision Strategy giúp decoupling (tách rời) logic chính sách khỏi tài nguyên.

## 2. Luồng nội bộ & Cơ chế cấp thấp (Internal Workflow & Low-level Mechanisms)

Decision Strategy được áp dụng ở hai cấp độ (level) khác nhau trong Keycloak Policy Evaluation Engine:
- **Permission Level:** Khi nhiều Policy được gán vào một Permission.
- **Policy Level:** Khi tạo ra một **Aggregated Policy** (Chính sách tổng hợp gồm nhiều chính sách con).

```mermaid
flowchart TD
    Req[Evaluation Request từ Client] --> Engine[Policy Evaluation Engine]
    Engine --> GetPolicies[Lấy danh sách Policies liên kết với Resource]
    
    GetPolicies --> EvalP1[Đánh giá Policy 1: TRUE]
    GetPolicies --> EvalP2[Đánh giá Policy 2: FALSE]
    GetPolicies --> EvalP3[Đánh giá Policy 3: TRUE]
    
    EvalP1 --> Strategy[Decision Strategy Resolver]
    EvalP2 --> Strategy
    EvalP3 --> Strategy
    
    Strategy --> CheckAff{AFFIRMATIVE?}
    CheckAff -- Yes --> OutputGrant[GRANT (Vì có 1 cái TRUE)]
    
    CheckAff -- No --> CheckUnanimous{UNANIMOUS?}
    CheckUnanimous -- Yes --> OutputDeny1[DENY (Vì có 1 cái FALSE)]
    
    CheckUnanimous -- No --> CheckConsensus{CONSENSUS?}
    CheckConsensus -- Yes --> Count{2 TRUE > 1 FALSE}
    Count -- Yes --> OutputGrant2[GRANT]
    Count -- No --> OutputDeny2[DENY]
    
    OutputGrant --> Token[Cấp RPT - Requesting Party Token]
    OutputGrant2 --> Token
```

**Cơ chế thực thi (Low-level):**
1. Khi Keycloak nhận yêu cầu lấy token `RPT`, nó duyệt qua cây Policy bằng thuật toán **Depth-First Search (DFS)**.
2. Tại mỗi Node, Engine sẽ tính toán và đếm kết quả của các Node con (`positiveVotes`, `negativeVotes`).
3. Dựa trên hằng số `DecisionStrategy` của Node đó, hàm đánh giá trả về trạng thái `Effect.PERMIT` hoặc `Effect.DENY`.
4. Nếu Keycloak sử dụng **UNANIMOUS**, nó hỗ trợ cơ chế *Short-Circuiting* (Đoản mạch). Tức là nếu đánh giá Policy đầu tiên trả về DENY, hệ thống sẽ dừng ngay lập tức không đánh giá các Policy còn lại để tiết kiệm CPU và truy vấn cơ sở dữ liệu.

## 3. Thực hành tốt nhất & Bảo mật (Best Practices & Security)

> [!WARNING]
> **Rủi ro rò rỉ quyền (Over-privilege) với AFFIRMATIVE:** Mặc định Keycloak set Decision Strategy là Affirmative. Nếu bạn có một chính sách yêu cầu "Chỉ VIP", nhưng bạn lại lỡ thêm nhầm một chính sách trống (Always True) vào Permission, thì với Affirmative, mọi người dùng đều có quyền (GRANT). Luôn kiểm tra kỹ các Policy liên kết.

> [!IMPORTANT]
> **Thiết kế Zero Trust:** Trong môi trường Enterprise, hãy ưu tiên sử dụng chiến lược **UNANIMOUS**. Mọi yêu cầu truy cập tài nguyên nhạy cảm phải đồng thời thỏa mãn: đúng User, đúng thời gian, đúng IP, và không bị khóa tài khoản.

- **Dùng Aggregated Policies thay vì gộp logic:** Thay vì viết một JavaScript Policy rất dài chứa nhiều điều kiện `if/else`, hãy tách chúng thành các Policy nhỏ (VD: `IP Policy`, `Time Policy`) và kết hợp chúng bằng Aggregated Policy với chiến lược UNANIMOUS. Điều này giúp dễ tái sử dụng và kiểm thử.

## 4. Cấu hình minh họa thực tế (Configuration Examples)

**Yêu cầu cấu hình:** Cấp quyền xóa dữ liệu nhạy cảm (Delete_Resource). Quyền này chỉ dành cho người có chức vụ **Admin** VÀ phải sử dụng **Mạng Nội bộ (IP nội bộ)**.

1. **Tạo Role Policy:**
   - Name: `Require-Admin-Role`
   - Realm Roles: `admin`
2. **Tạo JavaScript Policy (IP Check):**
   - Name: `Require-Internal-IP`
   - Code: 
     ```javascript
     var context = $evaluation.getContext();
     var identity = context.getIdentity();
     var ip = context.getAttributes().getValue('X-Forwarded-For');
     if (ip && ip.asString(0).startsWith('10.0.0.')) {
         $evaluation.grant();
     } else {
         $evaluation.deny();
     }
     ```
3. **Tạo Permission (Kết nối Tài nguyên với Policy):**
   - Name: `Delete-Resource-Permission`
   - Resource: `CustomerData`
   - Scopes: `urn:app:scopes:delete`
   - Apply Policies: 
     - Chọn `Require-Admin-Role`
     - Chọn `Require-Internal-IP`
   - **Decision Strategy:** Đổi từ `Affirmative` thành **`UNANIMOUS`**.

Kết quả: Hệ thống yêu cầu bắt buộc (AND) phải thỏa mãn cả chức vụ Admin và đang truy cập từ lớp mạng `10.0.0.x`.

## 5. Trường hợp ngoại lệ (Edge Cases)

- **Hành vi hòa (Tie) trong CONSENSUS:** Khi số lượng Policy GRANT bằng số lượng Policy DENY, Keycloak sẽ từ chối quyền truy cập theo cơ chế fail-safe. Đừng bao giờ áp dụng Consensus nếu bạn có số lượng Policy chẵn mà không chắc chắn về trọng số của chúng.
- **Vòng lặp quyền (Policy Evaluation Loop):** Việc tạo các Aggregated Policy gọi vòng tròn lẫn nhau có thể gây ra hiện tượng Stack Overflow trên Server. Luôn giữ cấu trúc cây Policy một chiều (Directed Acyclic Graph - DAG).
- **Thiếu Policy (Empty Policy list):** Nếu một Permission được tạo ra mà không có bất kỳ Policy nào được liên kết, và Resource Server đang cấu hình Decision Strategy là Affirmative/Unanimous, thì mặc định Keycloak sẽ DENY (từ chối). Tức là "Không có luật nào cấp quyền = Không có quyền".

## 6. Câu hỏi Phỏng vấn (Interview Questions)

**Câu 1 (Junior):** Ý nghĩa của chiến lược AFFIRMATIVE trong Keycloak Authorization là gì?
*Đáp án:* Đây là chiến lược tương đương với phép toán logic `OR`. Khi nhiều chính sách áp dụng lên một tài nguyên, chỉ cần một chính sách trả về kết quả cho phép (GRANT), hệ thống sẽ cấp quyền ngay lập tức.

**Câu 2 (Junior):** Nếu muốn tất cả các điều kiện đều phải thỏa mãn mới cấp quyền, bạn chọn Decision Strategy nào?
*Đáp án:* Chọn `UNANIMOUS` (Đồng thuận). Tương đương với logic `AND`.

**Câu 3 (Senior):** Trong chiến lược UNANIMOUS, Keycloak có thực thi tối ưu hóa hiệu suất (Performance Optimization) nào khi đánh giá không?
*Đáp án:* Có. Keycloak áp dụng thuật toán **Short-Circuiting** (đoản mạch). Nếu trong danh sách 5 policies, policy thứ 2 trả về DENY, thì 3 policy còn lại sẽ không được đánh giá (không gọi database, không thực thi script) để tiết kiệm tài nguyên hệ thống, và trả về kết quả tổng là DENY ngay lập tức.

**Câu 4 (Senior):** Một Aggregated Policy (Policy tổng hợp) chứa Policy A và Policy B, Decision Strategy là AFFIRMATIVE. Aggregated Policy này lại được đưa vào một Permission cùng với Policy C, và Decision Strategy của Permission là UNANIMOUS. Điều gì xảy ra?
*Đáp án:* Quy trình đánh giá sẽ là: `(Policy A OR Policy B) AND (Policy C)`. Client chỉ được cấp quyền nếu Policy C thành công, và ít nhất một trong hai Policy A hoặc B cũng phải thành công.

**Câu 5 (Senior):** Bạn sẽ xử lý trường hợp một API yêu cầu "User phải có Role User hoặc Role Admin, nhưng bắt buộc không nằm trong danh sách Blacklist" bằng Keycloak như thế nào mà không code ở Backend?
*Đáp án:* 
- Tạo Role Policy 1: `Has-User-Role`
- Tạo Role Policy 2: `Has-Admin-Role`
- Tạo Group Policy 3: `Not-In-Blacklist` (Đảo ngược logic bằng thuộc tính Logic=Negative).
- Tạo Aggregated Policy 4 (Affirmative): Gộp Role 1 và Role 2.
- Gán Policy 4 và Policy 3 vào chung một Permission, đặt Permission đó là `UNANIMOUS`.

## 7. Tài liệu tham khảo (References)

- [Keycloak Authorization Services Guide - Decision Strategies](https://www.keycloak.org/docs/latest/authorization_services/#_policy_decision_strategies)
- [OAuth 2.0 / UMA Architecture Framework](https://docs.kantarainitiative.org/uma/wg/rec-oauth-uma-grant-2.0.html)
