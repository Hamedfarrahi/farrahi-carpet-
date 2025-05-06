// CarpetColorPickerApp.swift
import SwiftUI

@main
struct CarpetColorPickerApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// ContentView.swift
import SwiftUI
import UIKit
import ColorThiefSwift

struct ContentView: View {
    @State private var image: UIImage?
    @State private var colors: [UIColor] = []
    @State private var isPickerPresented = false

    var body: some View {
        VStack {
            if let image = image {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
                    .frame(height: 300)
            } else {
                Text("تصویری بارگذاری نشده است")
            }

            Button("بارگذاری تصویر فرش") {
                isPickerPresented.toggle()
            }
            .padding()
            .sheet(isPresented: $isPickerPresented) {
                ImagePicker(image: $image, onImagePicked: extractColors)
            }

            ScrollView(.horizontal) {
                HStack(spacing: 10) {
                    ForEach(colors, id: \.self) { color in
                        VStack {
                            Rectangle()
                                .fill(Color(color))
                                .frame(width: 60, height: 60)
                                .cornerRadius(8)
                            Text(rgbString(from: color))
                                .font(.caption)
                        }
                    }
                }.padding()
            }
        }
        .padding()
    }

    func extractColors(from image: UIImage) {
        if let palette = ColorThief.getPalette(from: image, colorCount: 8) {
            colors = palette.map { $0.makeUIColor() }
        }
    }

    func rgbString(from color: UIColor) -> String {
        var red: CGFloat = 0, green: CGFloat = 0, blue: CGFloat = 0
        color.getRed(&red, green: &green, blue: &blue, alpha: nil)
        return String(format: "RGB(%d, %d, %d)", Int(red * 255), Int(green * 255), Int(blue * 255))
    }
}

// ImagePicker.swift
import SwiftUI
import UIKit

struct ImagePicker: UIViewControllerRepresentable {
    @Binding var image: UIImage?
    var onImagePicked: (UIImage) -> Void

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

    class Coordinator: NSObject, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
        let parent: ImagePicker

        init(_ parent: ImagePicker) {
            self.parent = parent
        }

        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let uiImage = info[.originalImage] as? UIImage {
                parent.image = uiImage        
                parent.onImagePicked(uiImage)
            }
            picker.dismiss(animated: true)
        }
    }
}

// Extensions.swift
import SwiftUI
import UIKit

extension Color {
    init(_ uiColor: UIColor) {
        self.init(UIColor.RGBColorSpaceConverter(uiColor))
    }
}

extension UIColor {
    static func RGBColorSpaceConverter(_ color: UIColor) -> UIColor {
        var r: CGFloat = 0
        var g: CGFloat = 0
        var b: CGFloat = 0
        var a: CGFloat = 0
        color.getRed(&r, green: &g, blue: &b, alpha: &a)
        return UIColor(red: r, green: g, blue: b, alpha: a)
    }
}



