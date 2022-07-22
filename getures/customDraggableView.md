# Generic Draggaeble View

```Swift
//
//  DraggableView.swift
//  SwiftUICarosuel
//
//  Created by Lakshya  Verma on 09/07/22.
//

import SwiftUI



enum DragState {
    
    case inactive
    case pressing
    case dragging(translation: CGSize)
    
    var translation : CGSize {
        switch self {
        case .inactive:
            return .zero
        case .pressing:
            return .zero
        case .dragging(let translation):
            return translation
        }
    }
    
    var isPressing:Bool {
        switch self {
        case .inactive:
            return false
        case .pressing,.dragging:
            return true
        }
    }
}


struct DraggableView<Content>:View where Content:View {
    
    @GestureState private var dragState = DragState.inactive
    @State private var position = CGSize.zero
    
    var content: () -> Content
    var body: some View {
       content()
            .opacity(dragState.isPressing ? 0.5: 1.0)
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

struct DraggableView_Previews: PreviewProvider {
    static var previews: some View {
        DraggableView() {
            Image(systemName: "star.circle.fill")
        }
    }
}

```