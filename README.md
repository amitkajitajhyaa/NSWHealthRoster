import SwiftUI
import GoogleMobileAds

// APIService (No network dependencies, works only with local data)
class APIService {
    static let shared = APIService()

    // Simulate login with stored credentials in UserDefaults
    func login(staffLinkNumber: String, password: String) -> Bool {
        // Store credentials locally
        UserDefaults.standard.set(staffLinkNumber, forKey: "staffLinkNumber")
        UserDefaults.standard.set(password, forKey: "password")
        return true // Always returns true since it's a device-only app
    }

    // Simulate fetching roster from local storage
    func fetchRoster(for date: String) -> [RosterEntry] {
        // Fetch stored roster or simulate with dummy data
        return [
            RosterEntry(name: "John Doe", role: "Nurse", shift: "Morning"),
            RosterEntry(name: "Jane Smith", role: "Doctor", shift: "Evening")
        ]
    }
}

// RosterEntry (Local storage model)
struct RosterEntry: Identifiable, Codable {
    let id = UUID()
    let name: String
    let role: String
    let shift: String
}

// BannerAdView (for displaying the banner ad)
struct BannerAdView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        let viewController = UIViewController()
        let adView = GADBannerView(adSize: GADAdSizeBanner)
        adView.adUnitID = "ca-app-pub-3940256099942544/2934735716" // Test Ad Unit ID
        adView.rootViewController = viewController
        adView.load(GADRequest())
        viewController.view.addSubview(adView)
        
        adView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            adView.centerXAnchor.constraint(equalTo: viewController.view.centerXAnchor),
            adView.bottomAnchor.constraint(equalTo: viewController.view.bottomAnchor)
        ])
        
        return viewController
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {}
}

// Support View
struct SupportView: View {
    var body: some View {
        VStack {
            Text("Support").font(.largeTitle).padding()
            Text("For support, please contact us via email at support@yourapp.com.")
            Spacer()
        }
        .navigationTitle("Support")
    }
}

// Help View
struct HelpView: View {
    var body: some View {
        VStack {
            Text("Help").font(.largeTitle).padding()
            Text("Here you can find information on how to use the app.")
            Spacer()
        }
        .navigationTitle("Help")
    }
}

// Contact View
struct ContactView: View {
    var body: some View {
        VStack {
            Text("Contact").font(.largeTitle).padding()
            Text("You can reach us at contact@yourapp.com for inquiries.")
            Spacer()
        }
        .navigationTitle("Contact")
    }
}

// Donation View
struct DonationView: View {
    var body: some View {
        VStack {
            Text("Donation").font(.largeTitle).padding()
            Text("If you'd like to support the development of this app, please donate to our PayPal account: paypal@yourapp.com.")
            Spacer()
        }
        .navigationTitle("Donation")
    }
}

// Privacy Policy View
struct PrivacyPolicyView: View {
    var body: some View {
        ScrollView {
            VStack {
                Text("Privacy Policy").font(.largeTitle).padding()
                Text("We do not collect, share, or store any personal data. All data is stored locally on your device.")
                Spacer()
            }
            .padding()
        }
        .navigationTitle("Privacy Policy")
    }
}

// Login View (Modified for local data and no online request)
struct LoginView: View {
    @State private var staffLinkNumber = ""
    @State private var password = ""
    @State private var isLoggedIn = false

    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                Text("NSW Health Roster")
                    .font(.largeTitle)
                    .bold()
                    .padding()
                
                TextField("StaffLink Number", text: $staffLinkNumber)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .keyboardType(.numberPad)
                    .padding(.horizontal)
                
                SecureField("Password", text: $password)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .padding(.horizontal)
                
                Button(action: {
                    let success = APIService.shared.login(staffLinkNumber: staffLinkNumber, password: password)
                    if success {
                        isLoggedIn = true
                    }
                }) {
                    Text("Login")
                        .font(.headline)
                        .foregroundColor(.white)
                        .padding()
                        .frame(maxWidth: .infinity)
                        .background(Color.blue)
                        .cornerRadius(10)
                        .padding(.horizontal)
                }
                
                Spacer()
                
                NavigationLink("Forgot Password?", destination: HelpView())
                    .font(.footnote)
                    .foregroundColor(.blue)
                
                NavigationLink("Support", destination: SupportView())
                NavigationLink("Help", destination: HelpView())
                NavigationLink("Contact", destination: ContactView())
                NavigationLink("Donate", destination: DonationView())
                NavigationLink("Privacy Policy", destination: PrivacyPolicyView())
            }
            .padding()
            .fullScreenCover(isPresented: $isLoggedIn) {
                RosterView()
            }
        }
    }
}

// Roster View (Fetching local data)
struct RosterView: View {
    @State private var selectedDate = Date()
    @State private var rosterEntries: [RosterEntry] = []

    var formattedDate: String {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd"
        return formatter.string(from: selectedDate)
    }

    var body: some View {
        VStack {
            Text("Roster for \(formattedDate)")
                .font(.title2)
                .padding()

            DatePicker("Select Date", selection: $selectedDate, displayedComponents: .date)
                .datePickerStyle(GraphicalDatePickerStyle())
                .padding()
                .onChange(of: selectedDate) { _ in
                    rosterEntries = APIService.shared.fetchRoster(for: formattedDate)
                }

            List(rosterEntries) { colleague in
                HStack {
                    VStack(alignment: .leading) {
                        Text(colleague.name).font(.headline)
                        Text(colleague.role).font(.subheadline)
                    }
                    Spacer()
                    Text(colleague.shift).font(.subheadline)
                }
            }

            BannerAdView()
        }
        .onAppear {
            rosterEntries = APIService.shared.fetchRoster(for: formattedDate)
        }
    }
}

// Main App Entry Point
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            LoginView()
        }
    }
}
