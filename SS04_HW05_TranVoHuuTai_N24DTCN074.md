# BÀI 5: Sáng tạo (Thiết kế Quy trình & Prompt Kiểm tra Giao dịch đáng ngờ)

**Họ và tên:** Trần Võ Hữu Tài
**Mã sinh viên:** N24DTCN074

## 1. Ý đồ thiết kế quy trình 2 bước
Quy trình thiết kế này nhắm tới việc tách biệt 2 giai đoạn vòng đời phần mềm: Lập trình (Code Generation) và Tối ưu (Optimization).
- **Bước 1:** Bắt AI tập trung giải quyết độ chính xác của luồng nghiệp vụ (Business Logic Accuracy). Code ở bước này sẽ dễ bị AI tiếp cận theo cách đơn giản nhất là duyệt lồng nhau $O(N^2)$ để đối chiếu cặp giao dịch.
- **Bước 2:** Bắt AI đổi vai trò thành Chuyên gia Tối ưu hiệu năng. Bằng cách chỉ ra rõ điểm nghẽn bộ nhớ $O(N^2)$ và định hướng cách tối ưu "gom nhóm (grouping) và sắp xếp (sorting)", AI sẽ cấu trúc lại thuật toán về ngưỡng $O(N \log N)$, đảm bảo hệ thống FinTech không bị tắc nghẽn khi khối lượng giao dịch phình to hàng chục nghìn records.

## 2. Prompt sinh mã nguồn (Bước 1)
```text
Hãy đóng vai là một Kỹ sư phần mềm Backend Java. Viết cho tôi một class `FraudDetector` chứa hàm `detectFraud(List<Transaction> transactions)` để phát hiện giao dịch đáng ngờ.
Cấu trúc class `Transaction` gồm các thuộc tính: `String transactionId`, `String cardId`, `double amount`, `String location`, `LocalDateTime timestamp`.
Quy tắc gắn cờ đáng ngờ (hàm trả về danh sách các `transactionId` đáng ngờ):
1. Bất kỳ giao dịch đơn lẻ nào có giá trị (amount) > 100,000,000 VND.
2. Nếu xuất hiện 2 giao dịch trên cùng một thẻ (cardId), ở 2 địa điểm khác nhau (location), và có khoảng chênh lệch thời gian giữa chúng dưới 10 phút. Cả 2 giao dịch này đều bị gắn cờ.
Yêu cầu: Hãy kiểm tra cẩn thận dữ liệu biên đầu vào (danh sách null, chuỗi null).
```

## 3. Prompt kiểm chứng chéo hiệu năng (Bước 2)
```text
Bây giờ, hãy đóng vai trò là Chuyên gia Tối ưu hóa Thuật toán. 
Đoạn code bạn vừa sinh ra dùng 2 vòng lặp lồng nhau nên độ phức tạp thời gian đang là O(N^2). Với quy mô xử lý hàng vạn giao dịch, code này sẽ tạo ra nút thắt cổ chai hiệu năng.
Yêu cầu: Hãy phân tích điểm nghẽn và viết lại mã nguồn tối ưu thuật toán này xuống ngưỡng O(N log N). Gợi ý: Hãy gom nhóm các giao dịch theo thuộc tính `cardId`, sau đó sắp xếp theo thời gian (`timestamp`) để chỉ cần so sánh các giao dịch lân cận nhau theo 1 vòng lặp tuyến tính (O(N)), thay vì duyệt chéo toàn bộ tập dữ liệu gốc.
```

## 4. Minh chứng thực tế (Log Chat Iterative Prompting)

### Lượt 1: Sinh mã nguồn ban đầu (Code có độ phức tạp O(N^2))
**Trợ lý AI:**
```java
import java.time.Duration;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class FraudDetector {
    public List<String> detectFraud(List<Transaction> transactions) {
        Set<String> suspiciousIds = new HashSet<>();
        
        if (transactions == null || transactions.isEmpty()) {
            return new ArrayList<>();
        }

        for (int i = 0; i < transactions.size(); i++) {
            Transaction t1 = transactions.get(i);
            
            // Quy tắc 1
            if (t1.getAmount() > 100_000_000) {
                suspiciousIds.add(t1.getTransactionId());
            }

            // Quy tắc 2: Duyệt vòng lặp lồng nhau O(N^2) gây chậm chạp
            for (int j = i + 1; j < transactions.size(); j++) {
                Transaction t2 = transactions.get(j);
                
                if (t1.getCardId() != null && t1.getCardId().equals(t2.getCardId()) 
                    && t1.getLocation() != null && t2.getLocation() != null 
                    && !t1.getLocation().equals(t2.getLocation())) {
                    
                    long minutesDiff = Math.abs(Duration.between(t1.getTimestamp(), t2.getTimestamp()).toMinutes());
                    if (minutesDiff < 10) {
                        suspiciousIds.add(t1.getTransactionId());
                        suspiciousIds.add(t2.getTransactionId());
                    }
                }
            }
        }
        
        return new ArrayList<>(suspiciousIds);
    }
}
```

### Lượt 2: Phân tích và Tối ưu (Độ phức tạp O(N log N))
**Người dùng:** *(Gửi Prompt Bước 2 ở trên)*

**Trợ lý AI (Chuyên gia Tối ưu):**
```text
Bạn phân tích hoàn toàn chính xác. Vòng lặp lồng nhau ở bản gốc có độ phức tạp O(N^2) vì nó bắt buộc mọi giao dịch phải kiểm tra chéo với tất cả giao dịch còn lại, kể cả những thẻ không hề liên quan.
Bằng cách gom nhóm (grouping) theo `cardId` và sắp xếp (sorting) thời gian, ta có thể giới hạn phép so sánh chỉ đối với các giao dịch của cùng một thẻ và kề nhau về mặt thời gian. Việc gom nhóm và duyệt chiếm O(N), sắp xếp mỗi nhóm chiếm tối đa O(K log K) (với K là số giao dịch mỗi thẻ). Tổng thời gian tiến gần đến O(N log N), vượt trội hoàn toàn so với O(N^2).

Dưới đây là mã nguồn tối ưu:
```java
import java.time.Duration;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

public class FraudDetector {

    public List<String> detectFraud(List<Transaction> transactions) {
        Set<String> suspiciousIds = new HashSet<>();

        if (transactions == null || transactions.isEmpty()) {
            return new ArrayList<>();
        }

        // Quy tắc 1: Duyệt tuyến tính O(N) tìm giao dịch > 100tr
        for (Transaction t : transactions) {
            if (t.getAmount() > 100_000_000) {
                suspiciousIds.add(t.getTransactionId());
            }
        }

        // Tối ưu Quy tắc 2: O(N log N)
        // Bước 1: Lọc dữ liệu hợp lệ và Gom nhóm theo thẻ (Grouping)
        Map<String, List<Transaction>> txByCard = transactions.stream()
                .filter(t -> t.getCardId() != null && t.getLocation() != null && t.getTimestamp() != null)
                .collect(Collectors.groupingBy(Transaction::getCardId));

        // Bước 2: Duyệt từng nhóm thẻ
        for (List<Transaction> cardTxs : txByCard.values()) {
            if (cardTxs.size() < 2) continue;

            // Sắp xếp các giao dịch theo dòng thời gian
            cardTxs.sort(Comparator.comparing(Transaction::getTimestamp));

            // Chỉ duyệt 1 vòng để so sánh các giao dịch kề sát nhau
            for (int i = 0; i < cardTxs.size() - 1; i++) {
                Transaction current = cardTxs.get(i);
                Transaction next = cardTxs.get(i + 1);

                if (!current.getLocation().equals(next.getLocation())) {
                    long minutesDiff = Math.abs(Duration.between(current.getTimestamp(), next.getTimestamp()).toMinutes());
                    if (minutesDiff < 10) {
                        suspiciousIds.add(current.getTransactionId());
                        suspiciousIds.add(next.getTransactionId());
                    }
                }
            }
        }

        return new ArrayList<>(suspiciousIds);
    }
}
```
```
