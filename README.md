# Qwick
System manage state and reactive

## State
 - `window.ref`: khai báo 1 state. Thường dùng ref cho giá trị nguyên thủy
 
    ```js
      const count = window.ref(0)
      console.log(count.value) // 0

     count.value++
     console.log(count.value) // 1
    ```
    
  - `window.reactive`: khai báo 1 state type object.
  
     ```js
      const obj = window.reactive({ count: 1 })
      
      console.log(obj.count) // 1
      
      obj.count = 2
      console.log(obj.count) // 2
     ```
     
## Reactivity
 - `window.watch`: theo dõi sự thay đổi của state để trigger 1 hành động gì đó
 
    ```js
    watch(
      ref: Ref,
      callback: function,
      options?: {
        immediate?: boolean //Thực hiện ngay lập tức khi khởi tạo
      }
    
    const count = window.ref(0)
    window.watch(count, () => console.log('count', count.value))
    
    count.value = 2
    // count 2
    ```
   
 - `window.deepChanges`: theo dõi 1 reactive trigger khi property reactive thay đổi
 
    ```js
     deepChanges(
      target: Reactive,
      callback: function,
      options?: {
        immediate?: boolean // Thực hiện ngay lặp tức khi khởi tạo,
        deps?: Array[key: string] // Chỉ trigger khi các key trong list thay đổi. Nếu không truyền mặc định trigger tất cả key khi được thay đổi
      }
      
      const obj = window.reactive({ count: 1 })
      
      window.deepChanges(obj, (key) => console.log(key, obj[key], 'kv'))
      
      obj.count = 2
      // count 2 'kv'
      
      obj.a = 3 // No trigger. Do key a chưa được khai báo trong reactive nên sẽ k thay đổi. Muốn track key dynamic thì dùng `Set`
      
      // Use deps options
      const data = window.reactive({ count: 2, name: 'A' })
      
      window.deepsChange(data, (key) => console.log(key, data[key], 'kv'), {
        deps: ['name']
      })
      
      data.name = 'N' // name N 'kv'
      data.count = 2 // No trigger
    ```
    
 - `window.set`: Set dyamic 1 key trong reactive. Khuyến khích nên dùng `Set` khi thay đổi 1 key trong reactive
  
    ```js
      const obj = window.reactive({ count: 1 })
      
      window.deepChanges(obj, (key) => console.log(key, obj[key], 'kv'))
      
      obj.count = 2
      // count 2 'kv'
      
      obj.a = 3 // No trigger
      
      window.set(obj, 'a', 3)
      // count 3 'kv'
    ```
    
## Component
  - Các component phải viết class và gán biến instance vào window. Component khi khởi tạo sẽ có 1 id định danh. Khi loader sẽ tự động tìm id closest trong dom.
    Nếu component là global thì mặc định instance là instanceName
    
    ```js
      <div id="a">
        <div x:init="Button#init"></div>
      </div>
      
      window.VM.get('a') -> Button {}
    ```
  
    ```js
      class Button {
        constructor() {
        }
      }
      
      window.Button = Button
    ```
   
   - Khai báo 1 component Global: `componentName:global#method_event?`. component dùng chung trong 1 view
    
## Loader
  - Store tự load component theo syntax: `componentName:global?#method_event?`
    
    ```js
      <div x:init="Button#handleClick_event"></div>
    ```
    
  - `x:init`: load component khi dom được xuất hiện trên dom. Sử dụng khi muốn lazyload
     ```js
        <div x:init="Button#handleClick_event"></div>
        
        // Khởi tạo component Button // vm = new window.Button()
        // call method handleClick với truyền tham số event // vm.handleClick(event)
     ```
     
   - `x:visible`: lazy load image
   
      ```js
        <img x:visible data-src="" />
      ```
      
## Event Handler
  - Đăng ký event theo syntax: `on:[eventName]:[opts]="componentName#method"`
  
  - Chú ý trong server build cần add eventName
  
    ```js
      <div on:click="Button#handleClick_event"></div>
      
      // Khi click sẽ call hàm handleClick trong instance Button
    ```
    
  - StopPropagation: Không trigger event bubbles
    ```js
      <div on:click on:click.stop></div>
    ```
   
  - useCapture: Capture event
    ```js
      <div on:click.capture></div>
    ```
   
  - props component:
    ```js
      <div on:click.props="data"></div> // Truyền data props tới component. Mặc định component sẽ call method beforeUpdate(data). Data là json
    ```
    
 ## Store
  - Chia sẽ state giữa các component. `useStore(storeName)`
  
  ```js
    const page = useStore('page')
    
    console.log(page.state)  // Reactive
    
    // update state by key
    page.dispatch('name', 'A')
    console.log(page.state.name) // 'A'
    
    // watch state. Bản chất sate trong store là 1 Reactive. Watch change giống với Reactive
    window.deepChanges(page.state, (key) => console.log(key, page.state[key], 'kv'))
  ```
