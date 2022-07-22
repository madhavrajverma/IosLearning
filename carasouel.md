#SwiftUI Carousel 

![](Carasouel.png)

 Card View

```swift 
import SwiftUI

struct CardView: View {
    
    var image: String
    var category: String
    var heading: String
    var author: String
    
    var body: some View {
        VStack {
            Image(image)
                .resizable()
                .aspectRatio(contentMode: .fit)
            
            HStack {
                VStack(alignment:.leading) {
                    Text(category)
                            .font(.headline)
                            .foregroundColor(.secondary)
                        Text(heading)
                            .font(.title)
                            .fontWeight(.black)
                            .foregroundColor(.primary)
                            .lineLimit(3)
                            .minimumScaleFactor(0.5)
                        Text(author.uppercased())
                            .font(.caption)
                    }
                    .foregroundColor(.secondary)
                Spacer()
            }.padding()
        }.cornerRadius(10)
            .overlay(
            RoundedRectangle(cornerRadius: 10)
                .stroke(Color(.sRGB, red: 150/255, green: 150/255, blue: 150/255,
                opacity: 0.1), lineWidth: 1)
            ).padding([.top,.horizontal])
    }
}
```

ContentView

```swift 
import SwiftUI

struct ContentView: View {
    var body: some View {
        ScrollView(.horizontal,showsIndicators: false) {
            VStack {
                HStack {
                        Group {
                            CardView(image: "swiftui-button", category: "SwiftUI", heading: "Drawing a Border with Rounded Corners", author: "Simon Ng")
                            CardView(image: "macos-programming", category: "macOS", heading: "Building a Simple Editing App", author: "Gabriel Theodoropoulos")
                            CardView(image: "flutter-app", category: "Flutter", heading: "Building a Complex Layout with Flutter", author: "Lawrence Tan")
                            CardView(image: "natural-language-api", category: "iOS", heading: "What's New in Natural Language API", author: "Sai Kambampati")
                        }.frame(width:300)
                }
                Spacer()
            }
            
        }
    }
}

```