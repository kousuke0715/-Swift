# -Swift
実家の散髪屋をより業務を減らすためにiosのアプリにより予約をアプリ内でおこなえるようにする。
# -コード全体の流れ
@main

└─KrsmaApp.swift

@Model

└─ Reservation.swift

@View

├─ ContentView.swift

└─ CutDetailSelectView.swift
# -コードを各部分に分けて紹介していく
# --アプリ本体
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
# --予約データー
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
# --日付け選択画面
struct ContentView: View {
    @Query private var records: [Reservation]
    @State private var selectedDate = Date()
# --選択した日時がようやく済みか確認
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
  # --予約状況
                if isBooked {
                    Text("この日時は予約済みです")
                        .foregroundStyle(.red)
                } else {
                    Text("この日時は予約できます")
                        .foregroundStyle(.green)
                }
# --予約可能な時だけ押せる
                NavigationLink {
                    CutDetailSelectView(
                        selectedDate: selectedDate
                    )
                } label: {
                    Text("この日時で予約")
                }
                .disabled(isBooked)
                Divider()
# --動作確認用:現在の予約一覧
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
# --名前・メニュー選択画面
struct CutDetailSelectView: View {
    // 前の画面から受け取った日時
    let selectedDate: Date
    @Environment(\.dismiss) private var dismiss
    @Environment(\.modelContext) private var modelContext
    @State private var name = ""
    @State private var selectedService = ""
    let services: [String] = [
        "カット大人：4000円",
        "カット子供：2500円",
        "パーマ：8000円",
        "ストレートパーマ：10000円",
        "カラー：7000円"
    ]
    var body: some View {
        VStack(spacing: 20) {
            Text("情報入力")
                .font(.title)
                .bold()
# --選んだ日付を表示
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
