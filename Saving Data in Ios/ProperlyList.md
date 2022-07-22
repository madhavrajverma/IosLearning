  # Property List Encoder and Deocder

```swift
    let taskPListURL = URL(fileURLWithPath: "PrioritizedTasks",
                           relativeTo: FileManager.documentsDirectoryURL).appendingPathExtension("plist")
```

```swift
  private func saveToPListTasks() {
        
        let encoder  = PropertyListEncoder()
        encoder.outputFormat = .xml
        
        do {
            let taskData = try encoder.encode(prioritizedTasks)
            try taskData.write(to: taskPListURL,options: .atomicWrite)
                
        }catch let error {
            print(error)
        }
    }
```
    
```swift  
    private func loadFromPListTasks() {
        
        guard FileManager.default.fileExists(atPath: taskPListURL.path) else {
            return
        }
        
        let deocder  = PropertyListDecoder()
        
        do {
            let tasksData = try Data(contentsOf: taskPListURL)
            let prioritizedTasks = try deocder.decode([PrioritizedTasks].self, from: tasksData)
            self.prioritizedTasks = prioritizedTasks
                
        }catch let error {
            print(error)
        }
    }

    ```