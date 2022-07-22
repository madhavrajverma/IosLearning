#Json Encoding and Decoding 

```swift
 let tasksJSONURL = URL(fileURLWithPath: "PrioritizedTasks",
                           relativeTo: FileManager.documentsDirectoryURL).appendingPathExtension("json")
```    

```swift
  private func loadJsonPriortizedTasks() {
        
        print(Bundle.main.bundleURL)
        print(FileManager.documentsDirectoryURL)
        let temporaryDirectoryURL = FileManager.default.temporaryDirectory
        print(temporaryDirectoryURL)
        
        //        guard let tasksJsonURL = Bundle.main.url(forResource: "PrioritizedTasks", withExtension: "json") else {
        //            return
        //        }
        
        print((try? FileManager.default.contentsOfDirectory(atPath: FileManager.documentsDirectoryURL.path)) ?? [])
        
        let decoder  = JSONDecoder()
        
        do {
            let tasksData = try Data(contentsOf: tasksJSONURL)
            let prioritizedTasks = try decoder.decode([PrioritizedTasks].self, from: tasksData)
            self.prioritizedTasks = prioritizedTasks
        }catch let error {
            print(error)
        }
        
    }
```

```swift
// Save 
     private func saveJSONPrioritizedTask() {
        
        let encoder  = JSONEncoder()
        encoder.outputFormatting = .prettyPrinted
        
        do {
            let taskData = try encoder.encode(prioritizedTasks)
            try taskData.write(to: tasksJSONURL,options: .atomicWrite)
                
        }catch let error {
            print(error)
        }
    }

```