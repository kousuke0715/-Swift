# Swift
実家の散髪店における電話予約対応の負担軽減を目的として、iOS向け予約アプリを開発。
## コード全体の流れ

@main

└─KrsmaApp.swift

@Model

└─ Reservation.swift

@View

├─ ContentView.swift

└─ CutDetailSelectView.swift
# コードを各部分に分けて紹介していく
## アプリ本体
```
import SwiftUI
import SwiftData
@main
struct KrsmaApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Reservation.self)
    }
}
```
## 予約データー型
```dateTime:Date```
にすることで予約日時を設定する型。
データの方には普段使う。
String → 文字列
Int    → 整数
Bool   → true / false
以外にも
Date   → 日付・時刻
がある。
```
@Model
class Reservation {
    var dateTime: Date
    var name: String
    var service: String
    init(dateTime: Date, name: String, service: String) {
        self.dateTime = dateTime
        self.name = name
        self.service = service
    }
}
```
# 日付け選択画面
## データの使用方針
```
struct ContentView: View {
    @Query private var records: [Reservation]
    @State private var selectedDate = Date()
```
## 選択した日時が予約済みか確認
```var isBooked:Bool```によりTrue,Falseで予約済みか判断する。
```records.contains```によりrecords内に条件に合うデータがあるのかを判断する。
```
Calendar.current.isDate(
                reservation.dateTime,
                equalTo: selectedDate,
                toGranularity: .minute
            )
```
により秒数は切り捨て合致するかどうかを判断する。
```
DatePicker(
                    "日時を選択",
                    selection: $selectedDate,
                    in: Date()...,
                    displayedComponents: [.date, .hourAndMinute]
                )
                .datePickerStyle(.graphical)
```
この部分ではカレンダーが表示されておりそこから予約日時を選択できる。
```.datePickerStyle(.graphical)```によりカレンダーを常に表示し続ける。
```
    var isBooked: Bool {
        records.contains { reservation in
            Calendar.current.isDate(
                reservation.dateTime,
                equalTo: selectedDate,
                toGranularity: .minute
            )
        }
    }
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                Text("予約日時を選択")
                    .font(.title)
                    .bold()
                DatePicker(
                    "日時を選択",
                    selection: $selectedDate,
                    in: Date()...,
                    displayedComponents: [.date, .hourAndMinute]
                )
                .datePickerStyle(.graphical)
```
  ## 予約状況
```isBooked```がTrue,Falseにより予約されているか判断し分かりやすく色を変えて表示している。
  ```
                if isBooked {
                    Text("この日時は予約済みです")
                        .foregroundStyle(.red)
                } else {
                    Text("この日時は予約できます")
                        .foregroundStyle(.green)
                }
```
## 予約可能な時だけ押せる
予約可能な場合に```CutDetailSelectView```を使い```selectedDate```を渡す。
```.disabled(isBooked)```により```isBooker```がTrueの場合押せなくなる。
```
                NavigationLink {
                    CutDetailSelectView(
                        selectedDate: selectedDate
                    )
                } label: {
                    Text("この日時で予約")
                }
                .disabled(isBooked)
                Divider()
```
## 動作確認用:現在の予約一覧
```
                Text("予約済み一覧")
                    .font(.headline)
                List {
                    ForEach(records) { reservation in
                        VStack(alignment: .leading) {
                            Text(reservation.name)
                                .font(.headline)
                            Text(reservation.service)
                            Text(
                                reservation.dateTime,
                                format: .dateTime
                                    .year()
                                    .month()
                                    .day()
                                    .hour()
                                    .minute()
                            )
                        }
                    }
                }
            }
            .padding()
            .navigationTitle("予約")
        }
    }
}
```
## 名前・メニュー選択画面
```let selectedDate:Date```でCutDetailSelectView```に渡して使用できるようにした。
```
struct CutDetailSelectView: View {
    // 前の画面から受け取った日時
    let selectedDate: Date
    @Environment(\.dismiss) private var dismiss
    @Environment(\.modelContext) private var modelContext
    @State private var name = ""
    @State private var selectedService = ""
    let services: [String] = [
         "調髪：4400円",
    "調髪顔剃り無し：4100円",
    "調髪のみ：3600円",
    "アイロン：4900円",
    "SPなし：4100円",
    "顔剃りSP：3900円",
    "SPセット：2600円",
    "顔剃り：2600円",
    "セット：1200円",
    "丸刈り：3300円",
    "女性顔剃り：3300円",
    "高校生調髪：3800円",
    "高校生カット：3600円",
    "スキンフェイド：6400円",
    "中学生調髪：3300円",
    "中学生丸刈り：2100円",
    "小学生調髪：2700円",
    "小学生丸刈り：1800円",
    "乳児：3100円",
    "パーマ：9400円〜",
    "パーマ染：15000円",
    "アイパー：8100円〜",
    "Sパーマ：13000円",
    "Sパーマ前：7900円",
    "Sパーマ染：15000円",
    "Cut 白髪染：6600円",
    "Cut カラー：7600円",
    "白髪染めのみ：4200円"
    ]
    var body: some View {
        VStack(spacing: 20) {
            Text("情報入力")
                .font(.title)
                .bold()
```
## 選んだ日付を表示
```
Text(
                selectedDate,
                format: .dateTime
                    .year()
                    .month()
                    .day()
                    .hour()
                    .minute()
            )
```
で読みやすい形に変えている。
```
 Text("メニューを選択")
                .font(.headline)
            List {
                ForEach(services, id: \.self) { service in
                    Button {
                        selectedService = service
                    } label: {
                        HStack {
                            Text(service)
                            Spacer()
                            if selectedService == service {
                                Image(systemName: "checkmark")
                            }
                        }
                    }
                }
            }
```
この部分でカットの選択肢をすべて出しそれをクリックすることで選択可能。
```
Button("予約完了") {
                let record = Reservation(
                    dateTime: selectedDate,
                    name: name,
                    service: selectedService
                )
                modelContext.insert(record)
                dismiss()
            }
```
予約完了ボタンを押すと```record```としてデータを保存する。
```
.disabled(
                name.isEmpty ||
                selectedService.isEmpty
            )
```
未選択であれば予約完了ボタンは押せない。
```
            Text(
                selectedDate,
                format: .dateTime
                    .year()
                    .month()
                    .day()
                    .hour()
                    .minute()
            )
            TextField("名前", text: $name)
                .textFieldStyle(.roundedBorder)
            Text("メニューを選択")
                .font(.headline)
            List {
                ForEach(services, id: \.self) { service in
                    Button {
                        selectedService = service
                    } label: {
                        HStack {
                            Text(service)
                            Spacer()
                            if selectedService == service {
                                Image(systemName: "checkmark")
                            }
                        }
                    }
                }
            }
            if selectedService != "" {
                Text("選択中：\(selectedService)")
            }
            Button("予約完了") {
                let record = Reservation(
                    dateTime: selectedDate,
                    name: name,
                    service: selectedService
                )
                modelContext.insert(record)
                dismiss()
            }
            .disabled(
                name.isEmpty ||
                selectedService.isEmpty
            )
        }
        .padding()
        .navigationTitle("予約内容")
    }
}
```
# 感想
今回のアプリ開発でまだデザインや効率的なコードが書けているのかは分かりません。反省点としては、macを持ってないこともあり実際にちゃんと動くのかとデザイン的には良いのかなどの部分はLLMにおかしくないか聞いては見ているのですが正直自信がない部分ではあります。なのでもっといろいろなアプリを作ってより高い技術を習得していこうと思います。
初めてのSwift学習からSwiftUI,SwiftDateまで学習することで自身の設定目標を達成に近くなったのは良いものの、Swift学習の工程でSwift自体はPaizaで学習しそのほかのSwiftUI,SwiftDateはやはりios端末がなくては難しいと感じました。ios端末ではVScodeも使用可能だそうです。なのでこれから研究や就活などで使うためのmacがあれば勉強を機器によってあきらめることは減るのではないかと思う。
