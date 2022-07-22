#Gestures in Swift

- Tap Gesture

```swift
struct TapGestureView :View {
    @State private var isPressed = false
    var body: some View {
        
            // MARK: - Tap Geture
            Image(systemName: "star.circle.fill")
                 .font(.system(size: 200))
                 .foregroundColor(.green)
                     .scaleEffect(isPressed ? 0.5:1.0)
                 .gesture(
                 TapGesture().onEnded{
                         withAnimation(.easeInOut) {
                             self.isPressed.toggle()
                         }
                     }
                )
    }
}

```

- Long Press gesture

```swift
struct LongPressGestureView : View {
    @State private var isPressed = false
    @GestureState private var longPressTap = false
    var body: some View {
        // MARK: - Long Press Gesture
        VStack {
            Text("Long Press gesture")
                .font(.title)
                .foregroundColor(.gray)
                .bold()
            Image(systemName: "star.circle.fill")
                 .font(.system(size: 200))
                 .opacity(longPressTap ? 0.4 : 1.0)
                 .foregroundColor(.green)
                 .scaleEffect(isPressed ? 0.5:1.0)
                 .gesture(
                    LongPressGesture(minimumDuration: 1.0)
                        .updating($longPressTap, body: { (currentState, state, transcation) in
                            state = currentState
                        })
                        .onEnded{ _ in
                            withAnimation(.easeInOut) {
                                self.isPressed.toggle()
                            }
                        }
                 )
            
        }
        
    }
}
```

- Drag Press gesture

```swift
struct DragGestureExample : View {
   
  
    @GestureState private var dragOffset = CGSize.zero
    @State private var position  = CGSize.zero
    
    var body: some View {
        VStack {
            VStack(spacing:30) {
                
            
                // MARK: - Drag Press Gesture
                VStack {
                    Text("Drag  gesture")
                        .font(.title)
                        .foregroundColor(.gray)
                        .bold()
                    
                    Image(systemName: "star.circle.fill")
                         .font(.system(size: 200))
                         .foregroundColor(.green)
                         .offset(x: position.width +  dragOffset.width, y: dragOffset.height + position.height)
                         .gesture(
                           DragGesture()
                            .updating($dragOffset, body: { value, state, transcation in
                                state = value.translation
                            })
                            .onEnded({ value in
                                self.position.height += value.translation.height
                                self.position.width += value.translation.width
                            })
                         )

                }

            }
            
            
        }
    }
}
```


- Combining gesture

```swift
struct SwiftUIGesture: View {    
    @GestureState private var dragState = DragState.inactive
    
    @State private var position  = CGSize.zero
    
    var body: some View {
        
        DraggableView {
            Text("Swift UI")
                .font(.largeTitle)
                .foregroundColor(.green)
        }
        VStack {
            VStack(spacing:30) {

                VStack {

                    Image(systemName: "star.circle.fill")
                         .font(.system(size: 200))
                         .opacity(dragState.isPressing ? 0.5: 1.0)
                         .foregroundColor(.green)
                         .offset(x: position.width +  dragState.translation.width, y: dragState.translation.height + position.height)
                         .gesture(

                            LongPressGesture(minimumDuration:1)
                            .sequenced(before: DragGesture())

                            .updating($dragState, body: { value, state, transcation in
                                switch value {
                                case.first(true) :
                                    state = .pressing

                                case .second(true, let drag):
                                    state = .dragging(translation: drag?.translation ?? .zero)
                                default:
                                    break
                                }

                            })
                            .onEnded({ value in
                                guard case .second(true,let drag?) = value else {
                                    return
                                }
                                self.position.height +=  drag.translation.height
                                self.position.width += drag.translation.width
                            })
                         )

                }

            }


        }
    }
}


```