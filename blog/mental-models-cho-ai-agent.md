---
title: '39 Mental Models cho AI Agent: Từ First Principles đến OODA Loop'
description: >-
  Phân tích sâu về cc-thinking-skills — bộ 39 mô hình tư duy (First Principles,
  OODA, Cynefin, Theory of Constraints...) đóng gói thành skill cho Claude Code.
  Nguồn gốc nhận thức học, cách áp dụng, và sự trung thực đáng kinh ngạc về
  benchmark.
pubDate: '2026-06-24'
lang: vi
tags:
  - mental-models
  - ai-agents
  - claude-code
  - critical-thinking
  - decision-making
concepts:
  - Mental Models
  - First Principles
  - OODA Loop
  - Cynefin Framework
  - Theory of Constraints
  - Second-Order Thinking
  - Pre-Mortem
  - Inversion
  - Bayesian Reasoning
  - Hypothesis-Differential Debugging
tools:
  - Claude Code
topics:
  - AI Agents
  - Development Practices
readTime: 12
---
Bạn đưa cho Claude Code một bug khó. Nó đọc code, suy nghĩ, rồi đề xuất một bản vá. Bản vá đúng — nhưng chỉ giải quyết triệu chứng. Hai tuần sau, bug tương tự quay lại ở chỗ khác. Lần này Claude lại vá, rồi lại vá. Nếu hỏi nó "tại sao cứ lặp lại?", câu trả lời thường là một phân luận hợp lý nhưng thiếu chiều sâu — đúng từng bước, nhưng không chạm gốc rễ.

Vấn đề không phải AI "ngu". Vấn đề là nó thiếu **khung tư duy có cấu trúc** — những phương pháp mà con người đã mài giũa hàng thế kỷ để suy luận, ra quyết định, và giải vấn đề. Từ First Principles của Aristotle đến OODA Loop của John Boyd, từ Theory of Constraints của Goldratt đến Cynefin của Dave Snowden — những framework này không phải "thủ thuật prompt" mà là **kỹ thuật nhận thức đã được kiểm chứng**.

Repo [cc-thinking-skills](https://github.com/tjboudreaux/cc-thinking-skills) của tjboudreaux làm đúng một việc: đóng gói 39 mô hình tư duy này thành skill cho Claude Code. Không phải prompt pack bán kèm lời hứa "khiến AI thông minh hơn" — mà là một bộ công cụ có cấu trúc, có quy trình, có ranh giới rõ ràng, và quan trọng nhất: **có đánh giá trung thực**.

Bài này đi sâu vào những mô hình tư duy cốt lõi nhất trong 39 cái, nguồn gốc nhận thức học của chúng, và cách chúng biến đổi cách AI agent — và cả con người — tiếp cận vấn đề.

## Mental Models: Từ "mô hình nhỏ về thực tại" đến nan hoa tư duy

Năm 1943, nhà tâm lý học người Scotland [Kenneth Craik](https://en.wikipedia.org/wiki/Mental_model) đưa ra một ý tưởng gây sốt: não bộ con người xây dựng những "mô hình nhỏ về thực tại" (small-scale models of reality) bên trong đầu, rồi dùng các mô hình này để **mô phỏng trước** sự kiện trước khi nó xảy ra. Bạn không cần đút tay vào lửa để biết nó nóng — mô hình tâm lý về lửa đã chạy mô phỏng và báo "đừng".

Ý tưởng này, sau này được Philip Johnson-Laird phát triển thành **theory of mental models** vào thập niên 1980, trở thành một trong những trụ cột của khoa học nhận thức hiện đại. Một mental model không phải kiến thức khô — nó là **bộ mô phỏng** bạn chạy trong đầu khi gặp tình huống mới.

Nhưng chỉ có một mô hình thì không đủ. Charlie Munger — phó chủ tịch Berkshire Hathaway và một trong những nhà đầu tư vĩ đại nhất — đã đẩy ý tưởng này lên một tầm khác. Ông gọi hệ thống của mình là ["latticework of mental models"](https://modelthinkers.com/mental-model/mungers-latticework) (nan hoa tư duy): không học thuộc một mô hình, mà lắp đặt nhiều mô hình từ nhiều lĩnh vực khác nhau — vật lý, sinh học, kinh tế học, tâm lý học — vào một "bộ khung" liên kết. Khi gặp vấn đề, bạn không dùng một mô hình, mà rút ra mô hình phù hợp nhất từ khung đó.

> "Bạn phải biết các mô hình lớn từ các ngành lớn. Nếu bạn chỉ dùng một hoặc hai mô hình mà bạn học được trong tâm lý học, bạn sẽ vặn vẹo thực tế cho khớp với mô hình của mình." — Charlie Munger

![Latticework of Mental Models — từ Craik (1943) đến Claude Code (2026)](/images/diagram-mental-models-lattice.png)

Repo cc-thinking-skills chính là phiên bản số hóa của latticework này — 39 mental model được phân thành 6 nhóm: Decision Making, Cognitive & Behavioral, Systems & Strategy, Problem Solving, Estimation & Risk, Product & Innovation. Mỗi model là một skill độc lập, có thể gọi theo tên.

## First Principles: Bóc tách đến chân lý tối tiểu

"Tên lửa đắt quá, không thể giảm." Đó là câu trả lời của tư duy tương tự (reasoning by analogy) — nhìn xem người khác làm thế nào rồi chấp nhận. Elon Musk, khi thành lập SpaceX, không chấp nhận. Ông hỏi: *nguyên liệu thô* của một tên lửa giá bao nhiêu? Nhôm, titan, đồng, sợi carbon — chỉ khoảng 2% giá bán. Phần còn lại là chi phí sản xuất, biên lợi nhuận, và chuỗi cung ứng mà chưa ai thách thức.

Đó là [First Principles Thinking](https://jamesclear.com/first-principles) — tư duy từ nguyên lý, có nguồn từ Aristotle hơn 2000 năm trước. Thay vì suy luận "người khác làm vậy nên mình cũng vậy", bạn bóc tách vấn đề đến những chân lý cơ bản nhất — những gì chắc chắn đúng bất kể cách tiếp cận — rồi xây lại giải pháp từ đó.

Skill `thinking-first-principles` trong repo đóng gói quy trình này thành 5 bước rõ ràng: liệt kê giả định → bóc tách chân lý → thách thức từng giả định → xây lại từ đầu → kiểm chứng với thực tế.

![First Principles Thinking — so sánh với tư duy tương tự, ví dụ SpaceX](/images/diagram-first-principles.png)

Điểm thú vị nhất của skill này không phải quy trình (về cơ bản là Socratic questioning) mà là phần **"When NOT to Use"** — khi nào KHÔNG dùng. Skill nói rõ: nếu giới hạn là vật lý thật (không phải quy ước), hãy chấp nhận và đi tiếp. Nếu đã có giải pháp chuẩn đang hoạt động tốt, đừng tái phát minh bánh xe. Trong incident, hãy xử lý nguyên nhân khả nghi nhất trước (Occam's Razor), dành First Principles cho lúc thiết kế lại sau khi đã ổn định.

Sự trung thực này — biết ranh giới của mỗi framework — là điều phân biệt cc-thinking-skills với các prompt pack khác.

## OODA Loop: Xoay vòng nhanh hơn tình huống thay đổi

Năm 1970s, Đại tá John Boyd — phi công chiến đấu và chiến lược gia của Không quân Mỹ — quan sát một sự thật đơn giản: trong không chiến, phi công chiến thắng không phải người có kế hoạch hoàn hảo nhất, mà người **xoay vòng** Observe → Orient → Decide → Act nhanh hơn đối thủ. Kế hoạch hoàn hảo đến muộn còn nguy hiểm hơn không có kế hoạch.

[OODA Loop](https://en.wikipedia.org/wiki/OODA_loop) của Boyd trở thành một trong những framework có ảnh hưởng nhất trong quân sự, kinh doanh, và đặc biệt — incident response trong DevOps.

Bốn pha của OODA nghe đơn giản, nhưng chi tiết quyết định:

**Observe** (Quan sát) — thu thập dữ liệu rẻ nhất, tín hiệu cao nhất: metrics, logs, alerts, diff gần nhất. Có time-box — đừng quan sát mãi.

**Orient** (Định hướng) — đây là pha quan trọng nhất và dễ sai nhất. Bạn ghép dữ liệu quan sát được với mô hình tâm lý hiện có, hình thành giả thuyết. Sai lầm phổ biến: khóa cứng vào giả thuyết đầu tiên. Skill ép bạn giữ ít nhất 2 giả thuyết đối thủ và để bằng chứng mới dịch chuyển bạn.

**Decide** (Quyết định) — chọn hành động với ~70% confidence. Nguyên tắc: 70% bây giờ tốt hơn 90% quá muộn — nhưng chỉ cho hành động có thể đảo ngược.

**Act** (Hành động) — thi hành rồi **ngay lập tức** quay lại Observe. Hành động tạo thông tin mới; đừng chờ nó "ổn định".

![OODA Loop — bốn pha cho tình huống biến động nhanh](/images/diagram-ooda-loop.png)

Skill `thinking-ooda` rất rõ về khi nào KHÔNG dùng: nếu tình huống tĩnh và có thời gian → phân tích kỹ càng hơn sẽ thắng. Nếu hành động không thể đảo ngược → 70% confidence là chưa đủ. Nếu có thể định vị nguyên nhân rẻ (đọc diff/log) → dùng hypothesis-differential debugging (phần tiếp theo) thay vì xoay vòng trong bóng tối.

## Cynefin: Bản chất vấn đề quyết định cách tiếp cận

Có một câu hỏi cơ bản mà nhiều người bỏ qua khi gặp vấn đề: *loại vấn đề này thuộc loại nào?* Cùng một cách tiếp cận sẽ hiệu quả ở một loại và gây họa ở loại khác.

[Dave Snowden](https://thecynefin.co/about-us/about-cynefin-framework/) tạo ra Cynefin (phát âm /kəˈnɛvɪn/, tiếng Wales nghĩa là "nơi chốn") vào năm 1999 tại IBM, phân loại mọi vấn đề vào một trong bốn — hoặc năm — miền dựa trên quan hệ nhân–quả:

![Cynefin — phân loại vấn đề theo quan hệ nhân–quả](/images/diagram-cynefin.png)

**Clear (Rõ ràng)** — nhân–quả ai cũng thấy. Cách tiếp cận: Sense → Categorize → Respond. Áp dụng best practice, chạy runbook, không cần phức tạp hóa. Ví dụ: restart service khi hết memory.

**Complicated (Phức tạp)** — nhân–quả biết được nếu có chuyên môn/phân tích. Cách tiếp cận: Sense → Analyze → Respond. Có nhiều lời giải đúng, cần chuyên gia để chọn. Ví dụ: chọn database cho hệ thống mới.

**Complex (Phức hợp)** — nhân–quả chỉ rõ khi nhìn lại (emergent). Không ai, kể cả chuyên gia, có thể dự đoán trước. Cách tiếp cận: Probe → Sense → Respond. Chạy thử nghiệm nhỏ, an toàn thất bại (safe-to-fail), khuếch đại những gì hoạt động. Ví dụ: tìm product-market fit.

**Chaotic (Hỗn loạn)** — không nhận biết được quan hệ nhân–quả, tình huống biến động. Cách tiếp cận: Act → Sense → Respond. Hành động ngay để ổn định, hiểu nguyên nhân sau. Ví dụ: outage đang xảy ra.

Bản chất của Cynefin không phải bảng phân loại — mà là **cảnh báo sự sai lệch** (mismatch). Sai lầm phổ biến nhất là xử lý vấn đề Complex như Complicated: lập kế hoạch chi tiết, nhưng kết quả vẫn bất ngờ. Hoặc xử lý Chaotic như Complex: chạy experiment trong lúc đang outage. Snowden nói rõ: *"Hệ thống phức hợp mang tính khuynh hướng, không nhân–quả — không thể dự đoán, chỉ có thể ảnh hưởng."*

Skill `thinking-cynefin` đóng vai trò **router** — sau khi phân loại, bạn chuyển sang skill tương ứng (OODA cho chaos, systems thinking cho complex, hypothesis-differential cho complicated). Đây là điểm khác biệt cốt lõi so với các "prompt pack": skill biết giới hạn của mình và chuyển hướng khi cần.

## Theory of Constraints: Một pipeline chỉ nhanh bằng mắt xích chậm nhất

Năm 1984, [Eliyahu Goldratt](https://en.wikipedia.org/wiki/Theory_of_constraints) xuất bản "The Goal" — một cuốn tiểu thuyết kinh doanh kể về một nhà quản lý nhà máy đang thất bại. Qua nhân vật Jonah (nhiều người tin là đại diện cho chính Goldratt), cuốn sách truyền tải một nguyên lý giản dị nhưng sâu sắc: **một hệ thống có throughput bị giới hạn chỉ có đúng một constraint** (điểm thắt cổ chai) quyết định tốc độ toàn hệ thống. Tối ưu bất cứ thứ gì khác là lãng phí — thậm chí phản tác dụng.

Nguyên lý này áp dụng trực tiếp vào software engineering. Giả sử pipeline giao phần mềm của team bạn gồm 5 khâu:

![Theory of Constraints — chỉ một bottleneck quyết định throughput toàn hệ thống](/images/diagram-theory-of-constraints.png)

Code Review là bottleneck: 100% utilized, queue 5 ngày, throughput chỉ 5/tuần. Các khâu khác đều dư capacity. Nếu bạn tối ưu Development (thêm dev, tăng tốc code), throughput toàn hệ thống **vẫn là 5/tuần** — vì Code Review không xử lý nhanh hơn. Work-in-progress (WIP) tăng, queue dài hơn, còn chậm hơn.

Goldratt đưa ra **Five Focusing Steps**: (1) Tìm constraint, (2) Vắt kiệt constraint (không tốn tiền — không để idle, tối ưu quy trình), (3) Phục vụ constraint (các khâu khác pacing theo bottleneck), (4) Mở rộng constraint (thêm người/thiết bị — chỉ sau bước 2-3), (5) Lặp lại vì constraint mới sẽ xuất hiện.

Skill `thinking-theory-of-constraints` nhấn mạnh một insight phản trực giác: **giữ developer bận rộn vượt quá capacity review là HẠI throughput**. Tăng WIP không giúp — nó tạo hàng đợi, context switching, và cuối cùng chậm hơn.

## Khi mọi thứ đều đúng — nhưng vẫn sai: Hypothesis-Differential Debugging

Trong 39 skill, chỉ một cái đạt gần mức "được chứng minh cải thiện accuracy" trong benchmark: `thinking-scientific-method` (hypothesis-differential debugging). Bản đánh giá gốc ghi +5.3 điểm phần trăm (p=0.061, n=150) — gần đạt ngưỡng ý nghĩa thống kê nhưng chưa qua. Bản replication riêng đạt +8.0pp (p=0.001) — ý nghĩa. Nhưng vì bản primary không pass gate, verdict cuối là DIRECTIONAL-NOT-REPLICATED. Tác giả công khai kết quả này thay vì giấu.

Skill này có lý do hiệu quả: nó giải quyết đúng **điểm yếu nhất** của AI agent khi debug — khuynh hướng đoán mò một nguyên nhân rồi vá.

Nguyên lý cốt lõi: khi triệu chứng có nhiều nguyên nhân khả dĩ, đừng đoán. Liệt kê 3–5 giả thuyết đối thủ, gọi tên observation rẻ nhất để phân biệt chúng, nói trước falsifier (điều nào sẽ loại giả thuyết), rồi kiểm tra theo thứ tự "khả năng × rẻ".

![Hypothesis-Differential Debugging — liệt kê giả thuyết, tìm observation rẻ nhất](/images/diagram-scientific-method.png)

Quy trình năm bước rất cụ thể:

**Bước 1: Liệt kê ≥3 giả thuyết đối thủ.** Mỗi giả thuyết gọi tên file/hàm/config cụ thể. Không "backend issue" hay "race condition ở đâu đó" — đó là đoán, không phải giả thuyết.

**Bước 2: Gọi observation rẻ nhất cho mỗi giả thuyết.** Đọc hàm mà stack trace chỉ ra, grep caller, so sánh 2 dòng log, check schema. Observation xấu: deploy canary, chạy experiment dài, bắt user chờ.

**Bước 3: Nói falsifier TRƯỚC khi nhìn.** "Nếu refresh() không đọc rotated token state → loại H1." Viết trước để không tự lừa mình bằng confirmation search.

**Bước 4: Xếp hạng theo khả năng × rẻ.** Bắt đầu từ observation rẻ nhất phân biệt top hypothesis, không phải investigation phức tạp nhất.

**Bước 5: Dừng khi định vị trực tiếp.** Một giả thuyết được bằng chứng trực tiếp hỗ trợ, các đối thủ bị loại → gọi tên file/hàm cần sửa và bằng chứng.

Skill thậm chí cảnh báo: *"Nếu chỉ nghĩ được 1 giả thuyết, bạn đang đoán mò. Ép bản thân liệt kê đối thủ trước khi đào sâu."* Đây là phần giá trị nhất — không phải công cụ tìm bug, mà là **chống thiên kiến xác nhận**.

## "Và rồi thì sao?" — Second-Order Thinking

[Howard Marks](https://fs.blog/second-order-thinking/), nhà đầu tư huyền thoại của Oaktree Capital, phân biệt hai cấp tư duy:

**Tư duy bậc nhất**: "Team chậm → thêm engineer." Đơn giản, hiển nhiên, và thường đúng — ở bậc một.

**Tư duy bậc hai**: "Thêm engineer → coordination overhead tăng phi tuyến (Brooks's Law) → quyết định chậm hơn → team có thể CÒN chậm hơn."

![Second-Order Thinking — câu trả lời hiển nhiên thường sai](/images/diagram-second-order.png)

Nguyên tắc: câu trả lời hiển nhiên cho "Nên làm gì?" thường sai vì nó bỏ qua hệ quả tiếp theo. Skill `thinking-second-order` yêu cầu hỏi "Rồi sao?" liên tục qua các horizon: immediate (request tiếp theo), next deploy (tuần tới), at scale (tháng+, 10× adoption). Dừng ở hiệu ứng đầu tiên thực sự thay đổi quyết định — đừng chế ra cascade thứ ba, thứ tư viễn vông.

Đây là skill chống lại một dạng sai lầm rất phổ biến trong engineering: fix bề mặt giải quyết triệu chứng nhưng tạo nợ kỹ thuật ở tầng sâu hơn.

## Inversion và Pre-Mortem: Nghĩ về thất bại trước khi nó xảy ra

Hai skill này bổ sung cho nhau — cùng giải quyết một vấn đề: con người (và AI) **thiên vị lạc quan** khi lập kế hoạch.

**Inversion** — nhà toán học Carl Jacobi thường nói "Invert, always invert" (Đảo ngược, luôn luôn đảo ngược). Thay vì hỏi "làm sao thành công?", hỏi "làm sao **chắc chắn thất bại**?". [Charlie Munger](https://jamesclear.com/inversion/) áp dụng triệt để: trong bài phát biểu tốt nghiệp Harvard 1986, thay vì nói cách thành công, ông nói cách **rước thêm phiền não vào đời** — và ngược lại, đó chính là cách tránh thất bại.

Skill `thinking-inversion` gói gọn: (1) nêu mục tiêu rõ, (2) liệt kê 10+ cách đảm bảo thất bại, (3) chuyển top 3–5 thành yêu cầu phòng tránh cụ thể. Ví dụ: "Lưu password plaintext" → "Dùng bcrypt/argon2 với work factor ≥12."

**Pre-Mortem** — nhà tâm lý học [Gary Klein](https://www.theuncertaintyproject.org/tools/pre-mortem) phát triển kỹ thuật này dựa trên "prospective hindsight": thay vì hỏi "có thể sai gì?", giả định dự án **đã thất bại** rồi lý giải tại sao. Nghiên cứu từ năm 1989 cho thấy prospective hindsight tăng khả năng phát hiện rủi ro thêm [khoảng 30%](https://strategicdecisionsolutions.com/premortem-method/) so với đánh giá rủi ro truyền thống.

Skill `thinking-pre-mortem` yêu cầu: *"Đã 6 tháng sau. Dự án thất bại thảm. Chuyện gì đã xảy ra?"* — viết ở thì quá khứ. Đây không phải dự đoán, mà là **giải thích** một thất bại — và não bộ (cả người lẫn AI) giỏi giải thích hơn dự đoán. Liệt kê 15+ lý do cụ thể (không generic), phân loại, ưu tiên, rồi thêm safeguard cho top rủi ro.

Điểm tinh tế: skill phân biệt rõ khi nào dùng cái nào. Cho feature/design có scope nhỏ → Inversion (nhanh, liệt kê). Cho plan/launch/decision lớn → Pre-Mortem (sâu hơn, narrative).

## Sự trung thực đáng kinh ngạc: Zero ELEVATE

Phần gây ấn tượng nhất về repo này không phải 39 skill — mà là **cách tác giả đánh giá chúng**.

Không giống hầu hết "AI prompt pack" tuyên bố làm AI thông minh hơn mà không bao giờ kiểm chứng, cc-thinking-skills xây một pipeline đánh giá có tên "Elevate-or-Kill": so sánh output có skill vs. placebo prompt, kiểm soát độ dài, yêu cầu replication độc lập. Kết quả:

> **Không một skill nào đạt verdict ELEVATE được replicate vững chắc.**

Skill mạnh nhất, `thinking-scientific-method`, ghi +5.3pp trong bản primary (p=0.061 — dưới ngưỡng 0.05) và +8.0pp trong replication (p=0.001 — đạt). Nhưng vì bản primary không pass gate, verdict cuối là DIRECTIONAL-NOT-REPLICATED. Tác giả đề xuất nghiên cứu tiếp theo với N lớn hơn, pre-registered.

Điều này **không** nghĩa skill vô dụng. Tác giả nói rõ: coi chúng như **khung lý luận có cấu trúc** (structured-reasoning scaffolds) dựa trên framework đã kiểm chứng — không phải lời hứa tăng accuracy. Framework thật và hữu ích; bằng chứng thực nghiệm về việc tăng accuracy cho model thì chưa đủ, và tác giả nói thẳng.

Trong một thế giới mà mọi repo AI đều quảng cáo "boost performance 10×", sự trung thực này quý hơn bất kỳ con số nào.

## Model Router: Không cần thuộc 39 cái

Bạn không cần nhớ 39 framework. Repo cung cấp `thinking-model-router` — một meta-skill đóng vai trò "bộ phận hướng dẫn". Bạn nói: *"Dùng thinking-model-router để chọn framework phù hợp cho vấn đề này."*

Router phân loại theo hai trục: **Domain** (Coding/Debugging, Architecture, Product, Business Strategy, Risk/Safety, Innovation...) và **Problem Type** (Diagnose, Decide, Understand, Create, Evaluate, Predict, Optimize). Từ đó, nó route đến skill phù hợp.

Ví dụ:
- Bug với nguyên nhân không rõ → Scientific Method, Five Whys Plus
- Hiệu năng giảm → Theory of Constraints, Systems Thinking
- Lỗi spans nhiều service → Systems Thinking, Feedback Loops
- Chọn công nghệ → Lindy Effect, Reversibility
- Microservices vs monolith → Cynefin, Reversibility

Router cũng biết khi nào **bỏ qua**: *"Nếu bạn đã biết model phù hợp, gọi trực tiếp — đừng route. Nếu không model nào rõ ràng giúp, cứ suy luận trực tiếp — đừng ép."*

## Kết: Cấu trúc tư duy, không phải phép thuật

39 mental model trong cc-thinking-skills không biến Claude Code thành siêu nhân. Benchmark nói rõ điều đó. Nhưng chúng cung cấp thứ quý giá hơn: **cấu trúc**.

Khi gặp bug khó, thay vì đoán mò, agent liệt kê giả thuyết đối thủ, nói falsifier trước, kiểm tra rẻ nhất trước. Khi ra quyết định kiến trúc, thay vì chọn theo cảm tính, agent hỏi: đây là one-way door hay two-way door? Khi lập kế hoạch, thay vì lạc quan, agent giả định thất bại rồi giải thích tại sao.

Đó không phải "prompt engineering" — đó là **kỹ thuật nhận thức** mà con người đã mài giũa từ Aristotle đến Munger, từ Boyd đến Klein. Việc đóng gói chúng thành skill cho AI agent không phải marketing, mà là sự tiếp nối tự nhiên: nếu mental model giúp con người tư duy tốt hơn, tại sao không cho AI?

Câu trả lời trung thực của repo: chúng giúp, nhưng chưa được chứng minh rõ ràng bằng số liệu. Và đó chính xác là lý do bạn nên dùng chúng — không phải vì lời hứa, mà vì **cấu trúc tư duy đúng luôn có giá trị**, kể cả khi benchmark chưa đủ mạnh để chứng minh.

---

**Tham khảo:**
- Repo: [tjboudreaux/cc-thinking-skills](https://github.com/tjboudreaux/cc-thinking-skills) (MIT License)
- [Kenneth Craik — Mental Models (Wikipedia)](https://en.wikipedia.org/wiki/Mental_model)
- [Charlie Munger's Latticework (ModelThinkers)](https://modelthinkers.com/mental-model/mungers-latticework)
- [First Principles Thinking (James Clear)](https://jamesclear.com/first-principles)
- [OODA Loop (Wikipedia)](https://en.wikipedia.org/wiki/OODA_loop)
- [Cynefin Framework (The Cynefin Co)](https://thecynefin.co/about-us/about-cynefin-framework/)
- [Theory of Constraints (Wikipedia)](https://en.wikipedia.org/wiki/Theory_of_constraints)
- [Second-Order Thinking (Farnam Street)](https://fs.blog/second-order-thinking/)
- [Pre-Mortem Technique (Gary Klein)](https://www.theuncertaintyproject.org/tools/pre-mortem)
- [Inversion (James Clear)](https://jamesclear.com/inversion/)
- [TRIZ (Wikipedia)](https://en.wikipedia.org/wiki/TRIZ)
- [Elevate-or-Kill Scorecard (repo)](https://github.com/tjboudreaux/cc-thinking-skills/blob/main/analysis/ELEVATE-OR-KILL-SCORECARD.md)
