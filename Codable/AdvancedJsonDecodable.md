# Advance JSON Decodable


## Decoding Unknown Keys

```swift 

let json = """
{
    "points": ["KSQL", "KWVI"],
    "KSQL": {
        "code": "KSQL",
        "name": "San Carlos Airport"
    },
    "KWVI": {
        "code": "KWVI",
        "name": "Watsonville Municipal Airport"
    }
}
""".data(using: .utf8)!


public struct Route: Decodable {
    public struct Airport: Decodable {
        public var code: String
        public var name: String
    }
    
    public var points: [Airport]
    
    private struct CodingKeys: CodingKey {
        var stringValue: String
        
        var intValue: Int? {
            return nil
        }
        
        init?(stringValue: String) {
            self.stringValue = stringValue
        }
        
        init?(intValue: Int) {
            return nil
        }
        
        static let points =
            CodingKeys(stringValue: "points")!
    }
    
    public init(from coder: Decoder) throws {
        let container = try coder.container(keyedBy: CodingKeys.self)
        
        var points: [Airport] = []
        let codes = try container.decode([String].self, forKey: .points)
        for code in codes {
            let key = CodingKeys(stringValue: code)!
            let airport = try container.decode(Airport.self, forKey: key)
            points.append(airport)
        }
        self.points = points
    }
}


let decoder = JSONDecoder()
let route = try decoder.decode(Route.self, from: json)
print(route.points.map{ $0.code })


```

##  Decoding Indeterminate Types

```swift
/// “Look — up in the sky! It’s a bird! It’s a plane! It’s... a   heterogeneous collection of Decodable elements!”
let json = """
[
    {
        "type": "bird",
        "genus": "Chaetura",
        "species": "Vauxi"
    },
    {
        "type": "plane",
        "identifier": "NA12345"
    }
]
""".data(using: .utf8)!

public struct Bird: Decodable {
    public var genus: String
    public var species: String
}

public struct Plane: Decodable {
    public var identifier: String
}


public enum Either<T, U> {
    case left(T)
    case right(U)
}

extension Either: Decodable where T: Decodable, U: Decodable {
    public init(from decoder: Decoder) throws {
        if let value = try? T(from: decoder) {
            self = .left(value)
        } else if let value = try? U(from: decoder) {
            self = .right(value)
        } else {
            let context = DecodingError.Context(
                codingPath: decoder.codingPath,
                debugDescription:
                "Cannot decode \(T.self) or \(U.self)"
            )
            throw DecodingError.dataCorrupted(context)
        }
    }
}


let decoder = JSONDecoder()
let objects = try! decoder.decode([Either<Bird, Plane>].self, from: json)

for object in objects {
    switch object {
    case .left(let bird):
        print("Poo-tee-weet? It's \(bird.genus) \(bird.species)!")
    case .right(let plane):
        print("Vooooooooooooooom! It's \(plane.identifier)!")
    }
}


```

## Decoding Arbitrary Types

```swift 

import Foundation

let json = """
{
    "title": "Sint Pariatur Enim ut Lorem Eiusmod",
    "body": "Cillum deserunt ullamco minim nulla et nulla ea ex eiusmod ea exercitation qui irure irure. Ut laboris amet Lorem deserunt consequat irure dolore quis elit eiusmod. Dolore duis velit consequat dolore. Qui aliquip ad id eiusmod in do officia. Non fugiat esse laborum enim pariatur cillum. Minim aliquip minim exercitation anim adipisicing amet. Culpa proident adipisicing labore enim ullamco veniam.",
    "metadata": {
        "key": "value"
    }
}
""".data(using: .utf8)!



public struct AnyDecodable: Decodable {
    public let value: Any
    
    public init(_ value: Any?) {
        self.value = value ?? ()
    }

    public init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        
        if container.decodeNil() {
            self.value = ()
        } else if let bool = try? container.decode(Bool.self) {
            self.value = bool
        } else if let int = try? container.decode(Int.self) {
            self.value = int
        } else if let uint = try? container.decode(UInt.self) {
            self.value = uint
        } else if let double = try? container.decode(Double.self) {
            self.value = double
        } else if let string = try? container.decode(String.self) {
            self.value = string
        } else if let array = try? container.decode([AnyDecodable].self) {
            self.value = array.map { $0.value }
        } else if let dictionary = try? container.decode([String: AnyDecodable].self) {
            self.value = dictionary.mapValues { $0.value }
        } else {
            throw DecodingError.dataCorruptedError(in: container, debugDescription: "AnyCodable value cannot be decoded")
        }
    }
}

public struct Report: Decodable {
    public var title: String
    public var body: String
    public var metadata: [String: AnyDecodable]
}


let decoder = JSONDecoder()
let report = try decoder.decode(Report.self, from: json)

print(report.title)
print(report.body)
print(report.metadata["key"])


```

## Decoding Multiple Representations

```swift 
import Foundation

let americanJSON = """
[
    {
        "fuel": "100LL",
        "price": 5.60
    },
    {
        "fuel": "Jet A",
        "price": 4.10
    }
]
""".data(using: .utf8)!

let canadianJSON = """
{
    "fuels": [
        {
            "type": "100LL",
            "price": 2.54
        },
        {
            "type": "Jet A",
            "price": 3.14,
        },
        {
            "type": "Jet B",
            "price": 3.03
        }
    ]
}
""".data(using: .utf8)!



public enum Fuel: String, Decodable {
    case jetA = "Jet A"
    case jetB = "Jet B"
    case oneHundredLowLead = "100LL"
}

extension Fuel: CustomStringConvertible {
    public var description: String {
        return self.rawValue
    }
}


public protocol FuelPrice {
    var type: Fuel { get }
    var pricePerLiter: Decimal { get }
    var currency: String { get }
}

extension FuelPrice {
    public var description: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currencyISOCode
        formatter.currencyCode = self.currency

        return "\(type.rawValue): \(formatter.string(from: self.pricePerLiter as NSNumber)!)"
    }
}

public struct CanadianFuelPrice: Decodable {
    public let type: Fuel
    
    /// CAD / liter
    public let price: Decimal
}

extension CanadianFuelPrice: FuelPrice {
    public var pricePerLiter: Decimal {
        return self.price
    }

    public var currency: String {
        return "CAD"
    }
}

extension CanadianFuelPrice: CustomStringConvertible {}


public struct AmericanFuelPrice: Decodable {
    public let fuel: Fuel
    
    /// USD / gallon
    public let price: Decimal
}

extension AmericanFuelPrice: FuelPrice {
    public var type: Fuel {
        return self.fuel
    }

    public var pricePerLiter: Decimal {
        return self.price /
                3.78541
    }

    public var currency: String {
        return "USD"
    }
}

extension AmericanFuelPrice: CustomStringConvertible {}


let decoder = JSONDecoder()
let usPrices = try! decoder.decode([AmericanFuelPrice].self, from: americanJSON)
let caPrices = try! decoder.decode([String: [CanadianFuelPrice]].self, from: canadianJSON)

var prices: [FuelPrice] = []
prices.append(contentsOf: usPrices)
prices.append(contentsOf: caPrices["fuels"]! as [FuelPrice])

print(prices)

```

## Decoding Inherited Types

```swift

import Foundation

let json = """
{
    "number": 7,
    "letter": "A",
    "mealPreference": "vegetarian"
}
""".data(using: .utf8)!

public class EconomySeat: Decodable {
    public var number: Int
    public var letter: String
    
    public init(number: Int, letter: String) {
        self.number = number
        self.letter = letter
    }
}

public class PremiumEconomySeat: EconomySeat {
    public var mealPreference: String?

    private enum CodingKeys: String, CodingKey {
        case mealPreference
    }
    
    public init(number: Int, letter: String, mealPreference: String?) {
        super.init(number: number, letter: letter)
        self.mealPreference = mealPreference
    }

    public required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.mealPreference =
            try container.decodeIfPresent(String.self, forKey: .mealPreference)
        try super.init(from: decoder)
    }
}


let decoder = JSONDecoder()
let seat = try! decoder.decode(PremiumEconomySeat.self, from: json)

print(seat.number)
print(seat.letter)
print(seat.mealPreference!)

```

## Decoding from Different Types of Values

```swift 
import Foundation

let json = """
{
    "coordinates": [
        {
            "latitude": 37.332,
            "longitude": -122.011
        },
        [-122.011, 37.332],
        "37.332, -122.011"
    ]
}
""".data(using: .utf8)!

import Foundation

public struct Coordinate: Decodable {
    public var latitude: Double
    public var longitude: Double
    public var elevation: Double?
    
    private enum CodingKeys: String, CodingKey {
        case latitude
        case longitude
        case elevation
    }
    
    public init(from decoder: Decoder) throws {
        if let container =
            try? decoder.container(keyedBy: CodingKeys.self)
        {
            self.latitude =
                try container.decode(Double.self, forKey: .latitude)
            self.longitude =
                try container.decode(Double.self, forKey: .longitude)
            self.elevation =
                try container.decodeIfPresent(Double.self,
                                              forKey: .elevation)
        } else if var container = try? decoder.unkeyedContainer() {
            self.longitude = try container.decode(Double.self)
            self.latitude = try container.decode(Double.self)
            self.elevation = try container.decodeIfPresent(Double.self)
        } else if let container = try? decoder.singleValueContainer() {
            let string = try container.decode(String.self)
            
            let scanner = Scanner(string: string)
            scanner.charactersToBeSkipped = CharacterSet(charactersIn: "[ ,]")

            var longitude = Double()
            var latitude = Double()
            
            
            guard scanner.scanDouble(&longitude),
                  scanner.scanDouble(&latitude)
                else {
                    throw DecodingError.dataCorruptedError(
                        in: container,
                        debugDescription: "Invalid coordinate string"
                    )
            }
            
            self.latitude = latitude
            self.longitude = longitude
            self.elevation = nil
        } else {
            let context = DecodingError.Context(codingPath: decoder.codingPath, debugDescription: "Unable to decode Coordinate")
            throw DecodingError.dataCorrupted(context)
        }
    }
}


let decoder = JSONDecoder()

let coordinates = try! decoder.decode([String: [Coordinate]].self, from: json)["coordinates"]
print(coordinates!)


```


##  Configuring How Encodable Types Are Represented

```swift 
import Foundation

let encoder = JSONEncoder()
encoder.userInfo[.colorEncodingStrategy] = ColorEncodingStrategy.hexadecimal(hash: true)

let cyan = Pixel(red: 0, green: 255, blue: 255)
let magenta = Pixel(red: 255, green: 0, blue: 255)
let yellow = Pixel(red: 255, green: 255, blue: 0)
let black = Pixel(red: 0, green: 0, blue: 0)



public struct Pixel {
    public var red: UInt8
    public var green: UInt8
    public var blue: UInt8

    public init(red: UInt8, green: UInt8, blue: UInt8) {
        self.red = red
        self.green = green
        self.blue = blue
    }
}

extension Pixel: Encodable {
    public func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        
        switch encoder.userInfo[.colorEncodingStrategy]
            as? ColorEncodingStrategy
        {
        case let .hexadecimal(hash)?:
            try container.encode(
                (hash ? "#" : "") +
                    String(format: "%02X%02X%02X", red, green, blue)
            )
        default:
            try container.encode(
                String(format: "rgb(%d, %d, %d)", red, green, blue)
            )
        }
    }
}


public enum ColorEncodingStrategy {
    case rgb
    case hexadecimal(hash: Bool)
}

extension CodingUserInfoKey {
    public static let colorEncodingStrategy =
        CodingUserInfoKey(rawValue: "colorEncodingStrategy")!
}

let json = try! encoder.encode([cyan, magenta, yellow, black])
let string = String(data: json, encoding: .utf8)!

print(string)

```