# Complex Json Payload Deocding

## json payload conatine nested data 
```swift 

let json = """
{
    "aircraft": {
        "identification": "NA12345",
        "color": "Blue/White"
    },
    "route": ["KTTD", "KHIO"],
    "departure_time": {
        "proposed": "2018-04-20T15:07:24-07:00",
        "actual": "2018-04-20T15:07:24-07:00"
    },
    "flight_rules": "IFR",
    "remarks": null
}
""".data(using: .utf8)!

public struct Aircraft: Codable {
    public var identification: String
    public var color: String
}

public enum FlightRules: String, Codable {
    case visual = "VFR"
    case instrument = "IFR"
}

import Foundation

public struct FlightPlan: Codable {    
    public var aircraft: Aircraft
    
    public var route: [String]
    
    public var flightRules: FlightRules
    
    private var departureDates: [String: Date]
    
    public var proposedDepartureDate: Date? {
        return departureDates["proposed"]
    }
    
    public var actualDepartureDate: Date? {
        return departureDates["actual"]
    }
    
    public var remarks: String?
    
    private enum CodingKeys: String, CodingKey {
        case aircraft
        case flightRules = "flight_rules"
        case route
        case departureDates = "departure_time"
        case remarks
    }
}

var decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

let plan = try! decoder.decode(FlightPlan.self, from: json)
print(plan.aircraft.identification)
print(plan.actualDepartureDate!)


```