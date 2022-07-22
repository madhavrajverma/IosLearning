
# Saving Data in Ios 

- Code Snippets

## 1 - Doucment Directory

```swift

// MARK: - Lecture - 1 documentDirectory

// Extension 

public extension FileManager {
   static var  documentDirectoryURL:URL {
       `default`.urls(for: .documentDirectory, in: .userDomainMask)[0]
    }
}


FileManager.documentDirectoryURL

```


 ## 2 - Paths

```swift

// MARK: - Lecture - 2 Paths

FileManager.documentDirectoryURL.path

let remiderDataURL = URL(fileURLWithPath: "Reminder", relativeTo: FileManager.documentDirectoryURL).path

let stringURL = FileManager.documentDirectoryURL.appendingPathComponent("String").appendingPathExtension("txt")

stringURL.path
 
 ```


 ## 3 - Challlenge URL

```swift
// MARK: - Lecture - 3 Challange Urls

let challengeString:String = "To do list"
let challengeURL:URL = URL(fileURLWithPath:challengeString,relativeTo:FileManager.documentDirectoryURL).appendingPathExtension("txt")

 challengeURL.lastPathComponent
```



## 4 - Data Types

```swift
// MARK: - Lecture - 4 Data Types


let age: UInt8 = 32
MemoryLayout.size(ofValue: age)
MemoryLayout<UInt8>.size
UInt8.min
UInt.max
let height: Int8 = 72
MemoryLayout.size(ofValue: height)
Int8.min
Int8.max

// Floats

let weight: Float = 154.5
MemoryLayout.size(ofValue: weight)
Float.leastNormalMagnitude
Float.greatestFiniteMagnitude

// Doubles

let earthRadius: Double = 3958.8
MemoryLayout.size(ofValue: earthRadius)
Double.leastNormalMagnitude
Double.greatestFiniteMagnitude

// Binary & Hexadecimal
let ageBinary: UInt8 = 0b0010_0000
let ageBinaryNegative: Int8 = -0b0010_0000
let weightHexadecimal: UInt16 = 0x9B
let weightHexadecimalNegative: Int16 = -0x9B

// Bytes
let favoriteBytes: [UInt8] = [
  240,          159,          152,          184,
  240,          159,          152,          185,
  0b1111_0000,  0b1001_1111,  0b1001_1000,  186,
  0xF0,         0x9F,         0x98,         187
]

MemoryLayout<UInt8>.size * favoriteBytes.count
```

 ## Saving & loading Data

```swift
// MARK: - Saving and Loading Data

let favoriteBytesData = Data(favoriteBytes)
let favoritesByteURL =  URL(fileURLWithPath: "Favorite Bytes", relativeTo: FileManager.documentDirectoryURL)
//print(favoritesByteURL.path)
try favoriteBytesData.write(to:favoritesByteURL)
let savedFavoritesBytesData = try Data(contentsOf:favoritesByteURL)
let savedFavoritesBytes = Array(savedFavoritesBytesData)

favoriteBytes ==  savedFavoritesBytes
favoriteBytesData == savedFavoritesBytesData
```


 ## Strings

```swift 

// MARK: - Strings
try favoriteBytesData.write(to:favoritesByteURL.appendingPathExtension("txt"))
let string = String(data:savedFavoritesBytesData, encoding: .utf8)!
```



## Challenge Strings

```swift
// MARK: - Challenge Strings
let catsURL = URL(fileURLWithPath:"Cats",relativeTo: FileManager.documentDirectoryURL)

try string.write(to: catsURL,atomically:true,encoding:.utf8)
let catsChallengeString = try String(contentsOf:catsURL)
```