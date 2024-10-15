# SwiftUI for React Developers

This is a cheat sheet to help React developers get started with SwiftUI.

## Basics

Building the Contents

One common part of these declarative UI frameworks is a DSL syntax.
Both React and SwiftUI provide a special inline syntax for building markup content.
React uses a syntax called [JSX](https://react.dev/learn/writing-markup-with-jsx) that's compiled to browser-compatible JavaScript. SwiftUI in contrast is compiled and utilizes directives and decorators like `@main` to work.

### Function Builders

In React:

```javascript
function Hello() {
  return (
    <div>
      <p>Hello</p>
      <p>React is awesome!</p>
    </div>
  );
}
```

In SwiftUI:

```swift
struct Hello: View {
    var body: some View {
        VStack {
            Text("Hello")
            Text("SwiftUI is awesome!")
        }
    }
}
```


### Props

In both React and SwiftUI, components are rendered and rerendered if their properties change.

In React:

```javascript
function Hello({ name }) {
  return <p>Hello, {name}!</p>;
}
```

In SwiftUI:

```swift
struct Hello: View {
    let name: String

    var body: some View {
        Text("Hello, \(name)!")
    }
}
```

### Conditionals and Lists

Structure of the contents can be dynamic, the most common patterns are conditional
and list.

In React:

.. code-block:: javascript

  const UserList = ({ users }) => {
    if (!users.length) {
      return <p>No users</p>;
    }

    return (
      <ul>
        {users.map(e => (
          <li key={e.id}>{e.username}</li>
        ))}
      </ul>
    );
  }

In SwiftUI:

.. code-block:: swift

  struct UserList: View {
      let users: [User]

      var body: some View {
          Group {
              if users.isEmpty {
                  Text("No users")
              } else {
                  VStack {
                      ForEach(users, id: \.id) {
                          Text("\($0.username)")
                      }
                  }
              }
          }
      }
  }

SwiftUI has built-in ``ForEach`` element, you don't need to manually map the data
array to views, so you can have a much neater code.

Events Handling
---------------
In React:

.. code-block:: javascript

  const Hello = () => {
    const clickHandler = useCallback(e => {
      console.log('Yay, the button is clicked!');
    }, []);
    return <button onClick={clickHandler}>Click Me</button>;
  };

In SwiftUI:

.. code-block:: swift

  struct Hello: View {
      var body: some View {
          Button("Click Me") {
              print("Yay, the button is clicked!")
          }
      }
  }

SwiftUI looks cleaner because there is no ``useCallback`` meme. In JavaScript, if
you create a function inside another function (let's say ``foo``), the former
always has a different reference every time ``foo`` is called. That means, the
component receives the function as a **prop** will be rerendered every time.

In consideration of performance, React provided ``useCallback``. It takes a value
as **dependency**, and will return the same reference if the dependency is not
changed.

In SwiftUI, Apple have not provided such mechanism, and developers can just take
no account of that.

State
-----
Sometimes, a component may retain some internal state even it's get updated by new
props. Or it need to update itself without the props changed. State was born for
this mission.

The example combines all the things we've talked above. Let's create a simple
counter.

In React:

.. code-block:: javascript

  const Counter = ({ initialValue }) => {
    const [counter, setCounter] = useState(initialValue);
    const increaseCounter = useCallback(() => {
      setCounter(counter + 1);
    }, [counter]);

    return (
      <div>
        <p>{counter}</p>
        <button onClick={increaseCounter}>Increase</button>
      </div>
    );
  };

In SwiftUI:

.. code-block:: swift

  struct Counter: View {
      let initialValue: Int

      @State
      var counter: Int

      init(initialValue: Int) {
          self.initialValue = initialValue
          _counter = State(initialValue: initialValue)
      }

      var body: some View {
          VStack {
              Text("\(counter)")
              Button("Increase") {
                  self.counter += 1
              }
          }
      }
  }

It seems to be a little complicated, let's decompose them into pieces.

The counter has a internal state: ``counter``, and it's initial value is from the
input props. In SwiftUI, a state is declared with ``@State`` property wrapper.
I'll explain that later but now, you could just consider it as a special mark.

The real ``counter`` value is wrapped in the ``_counter`` member variable (which
has type of ``State<Int>``), and we can use the input prop ``initialValue`` to
initialize it.

We trigger an update by directly setting the ``counter`` value. This is not just
an assignment, instead, this will cause some logic inside ``State`` to take effect
and notify the SwiftUI framework to update our view. SwiftUI packed the ``xxx``
and ``setXXX`` functions into this little syntactic sugar to simplify our code.

Effects
-------
How can we perform some side-effects when the component is updated? In React, we
have ``useEffect``:

.. code-block:: javascript

  const Hello = ({ greeting, name }) => {
    useEffect(() => {
      console.log(`Hey, ${name}!`);
    }, [name]);

    useEffect(() => {
      console.log('Something changed!');
    });

    return <p>{greeting}, {name}!</p>;
  };

In SwiftUI:

.. code-block:: swift

  func uniqueId() -> some Equatable {
      return UUID().uuidString  // Maybe not so unique?
  }

  struct Hello: View {
      let greeting: String
      let name: String

      var body: some View {
          Text("\(greeting), \(name)!")
              .onChange(of: name) { name in
                  print("Hey, \(name)!")
              }
              .onChange(of: uniqueId()) { _ in
                  print("Something changed!")
              }
      }
  }

In SwiftUI, we have neither hook functions nor lifecycle functions, but we have
modifiers! Every view type has a lot of modifier functions attached to it.

``onChange`` behaves just like ``useEffect``, the ``action`` closure is called
every time the ``value`` changes and the first time the receiver view renders.
But we must pass a value, if you need perform something whenever something
changed, you can use a trick:

Create a function that returns an unique object every time it gets called. You can
use **UUID**, global incrementing integer and even timestamps!

Lifecycle Callbacks
-------------------
In React:

.. code-block:: javascript

  const Hello = () => {
    useEffect(() => {
      console.log('I\'m just mounted!');
      return () => {
        console.log('I\'m just unmounted!');
      };
    }, []);

    return <p>Hello</p>;
  };

In SwiftUI:

.. code-block:: swift

  struct Hello: View {
      var body: some View {
          Text("Hello")
              .onAppear {
                  print("I'm just mounted!")
              }
              .onDisappear {
                  print("I'm just unmounted!")
              }
      }
  }

It's that easy.

Refs
----
Components can have some internal state that will not trigger view update when it
is changed. In React, we have **ref**:

In React:

.. code-block:: javascript

  const Hello = () => {
    const timerId = useRef(-1);
    useEffect(() => {
      timerId.current = setInterval(() => {
        console.log('Tick!');
      }, 1000);
      return () => {
        clearInterval(timerId.current);
      };
    });

    return <p>Hello</p>;
  };

In SwiftUI:

.. code-block:: swift

  struct Hello: View {
      private class Refs: ObservableObject {
          var timer: Timer?
      }

      @StateObject
      private var refs = Refs()

      var body: some View {
          Text("Hello")
              .onAppear {
                  refs.timer =
                      Timer.scheduledTimer(withTimeInterval: 1,
                                          repeats: true) { _ in
                          print("Tick!")
                      }
              }
              .onDisappear {
                  refs.timer?.invalidate()
              }
      }
  }

And we've got two approaches:

.. code-block:: swift

  struct Hello: View {
      @State
      private var timer: Timer? = nil

      var body: some View {
          Text("Hello")
              .onAppear {
                  self.timer =
                      Timer.scheduledTimer(withTimeInterval: 1,
                                          repeats: true) { _ in
                          print("Tick!")
                      }
              }
              .onDisappear {
                  self.timer?.invalidate()
              }
      }
  }

You may wonder why setting the state will not lead to view updates. SwiftUI is
pretty clever to handle the state, it uses a technique called
**Dependency Tracking**. If you are familiar with **Vue.js** or **MobX**, you may
understand it immediately. That's say, if we never **access** the state's value in
the view's building process (which not includes ``onAppear`` calls), that state
will be unbound and can be updated freely without causing view updates.

DOM Refs
--------
Accessing the native DOM object is an advanced but essential feature for Web
frontend development.

In React:

.. code-block:: javascript

  const Hello = () => {
    const pEl = useRef();
    useEffect(() => {
      pEl.current.innerHTML = '<b>Hello</b>, world!';
    }, []);

    return <p ref={pEl}></p>;
  };

In SwiftUI, we apparently don't have DOM, but for native applications, **View** is
a common concept. We can bridge native views to SwiftUI and gain control of them by
the way.

First, let's bridge an existed ``UIView`` to SwiftUI:

.. code-block:: swift

  struct MapView: UIViewRepresentable {
      let mapType: MKMapType
      let ref: RefBox<MKMapView>

      typealias UIViewType = MKMapView

      func makeUIView(context: Context) -> MKMapView {
          return MKMapView(frame: .zero)
      }

      func updateUIView(_ uiView: MKMapView, context: Context) {
          uiView.mapType = mapType
          ref.current = uiView
      }
  }

Every time we modified the input props, the ``updateUIView`` gets called, we can
update our ``UIView`` there. To export the ``UIView`` instance to the outer, we
declare a ref prop, and set it's ``current`` property to the view instance
whenever the ``updateUIView`` gets called.

Now we can manipulate the native view in our SwiftUI views:

.. code-block:: swift

  struct Hello: View {
      @State
      var mapType = MKMapType.standard

      @StateObject
      var mapViewRef = RefBox<MKMapView>()

      var body: some View {
          VStack {
              MapView(mapType: mapType, ref: mapViewRef)
              Picker("Map Type", selection: $mapType) {
                  Text("Standard").tag(MKMapType.standard)
                  Text("Satellite").tag(MKMapType.satellite)
                  Text("Hybrid").tag(MKMapType.hybrid)
              }
              .pickerStyle(SegmentedPickerStyle())
          }
          .onAppear {
              if let mapView = self.mapViewRef.current {
                  mapView.setRegion(.init(center: .init(latitude: 34, longitude: 108),
                                          span: MKCoordinateSpan(latitudeDelta: 50,
                                                                 longitudeDelta: 60)),
                                    animated: true)
              }
          }
      }
  }

Note that, we'd better encapsulate all the manipulations of native views to a
dedicated SwiftUI view. It's not a good practice to manipulate native objects
everywhere, as well as in React.

Context
-------
Passing data between the components can be hard, especially when you travel
through the hierachy. And **Context** to the rescue!

Let's look at an example in React:

.. code-block:: javascript

  const UserContext = createContext({});

  const UserInfo = () => {
    const { username, logout } = useContext(UserContext);
    if (!username) {
      return <p>Welcome, please login.</p>;
    }
    return (
      <p>
        Hello, {username}.
        <button onClick={logout}>Logout</button>
      </p>
    );
  }

  const Panel = () => {
    return (
      <div>
        <UserInfo />
        <UserInfo />
      </div>
    );
  }

  const App = () => {
    const [username, setUsername] = useState('cyan');
    const logout = useCallback(() => {
      setUsername(null);
    }, [setUsername]);
    return (
      <UserContext.Provider value={{ username, logout }}>
        <Panel />
        <Panel />
      </UserContext.Provider>
    );
  }

Even if the ``<UserInfo>`` is at a very deep position, we can use context to grab
the data we need through the tree. And also, contexts are often used by components
to communicate with each other.

In SwiftUI:

.. code-block:: swift

  class UserContext: ObservableObject {
      @Published
      var username: String?

      init(username: String?) {
          self.username = username
      }

      func logout() {
          self.username = nil
      }
  }

  struct UserInfo: View {
      @EnvironmentObject
      var userContext: UserContext

      var body: some View {
          Group {
              if userContext.username == nil {
                  Text("Welcome, please login.")
              } else {
                  HStack {
                      Text("Hello, \(userContext.username!).")
                      Button("Logout") {
                          self.userContext.logout()
                      }
                  }
              }
          }
      }
  }

  struct Panel: View {
      var body: some View {
          VStack {
              UserInfo()
              UserInfo()
          }
      }
  }

  struct App: View {
      @StateObject
      var userContext = UserContext(username: "cyan")

      var body: some View {
          VStack {
              Panel()
              Panel()
          }
          .environmentObject(userContext)
      }
  }

Contexts are provided by ``environmentObject`` modifier and can be retrieved via
``@EnvironmentObject`` property wrapper. And in SwiftUI, context objects can use
to update views. We don't need to wrap some functions that modifies the provider
into the context objects. Context objects are ``ObservableObject``, so they can
notify all the consumers automatically when they are changed.

Another interesting fact is that the contexts are identified by the type of
context objects, thus we don't need to maintain the context objects globally.

Implementations Behind the Scene
================================

What are ``View`` objects?
--------------------------
In SwiftUI, the ``View`` objects are different from the ``React.Component`` objects.
Actually, there is no ``React.Component`` equivalent in SwiftUI. ``View`` objects
are stateless themselves, they are just like ``Widget`` objects in Flutter, which
are used to describe the configuration of views.

That means, if you want attach some state to the view, you must mark it using
``@State``. Any other member variables are transient and live shorter than the view.
After all, ``View`` objects are created and destroyed frequently during the building
process, but meanwhile views may keep stable.

How ``@State`` works?
---------------------
To explain this question, you should know what is ``property wrapper`` before.
This proposal describe that in detail: `[SE-0258] Property Wrappers`_.

Before the ``View`` is mounted, SwiftUI will use type metadata to find out all the
``State`` fields (backends of the properties marked with ``@State``), and add them
to a ``DynamicPropertyBuffer`` sequentially, we call this process as "registration".

The buffer is aware of the view's lifecycle. When a new ``View`` object is created,
SwiftUI enumerates the ``State`` fields, and get its corresponding previous value
from the buffer. These fields are identified by their storage index in container
struct, pretty like how **Hook** works in React.

In this way, even though the ``View`` objects are recreated frequently, as long as
the view is not unmounted, the state will be kept.

How function builders works?
----------------------------
As we mention earlier, SwiftUI use **Function Builders** as DSL to let us build
contents. There is also a draft proposal about it: `Function builders (draft proposal)`_.

Let's first take a look at how JSX is transpiled to JavaScript. We have this:

.. code-block:: javascript

  const UserInfo = ({ users }) => {
    if (!users.length) {
      return <p>No users</p>;
    }

    return (
      <div>
        <p>Great!</p>
        <p>We have {users.length} users!</p>
      </div>
    );
  }

And this is the output from Babel with ``react`` preset:

.. code-block:: javascript

  const UserInfo = ({
    users
  }) => {
    if (!users.length) {
      return /*#__PURE__*/React.createElement("p", null, "No users");
    }

    return /*#__PURE__*/React.createElement("div", null,
      /*#__PURE__*/React.createElement("p", null, "Great!"),
      /*#__PURE__*/React.createElement("p", null, "We have ", users.length, " users!")
    );
  };

Most of the structure is identical, and the HTML tags are transformed to ``React.createElement``
calls. That makes sense, the function doesn't produce component instances, instead,
it produces elements. Elements describe how to configure components or DOM elements.

Now, let's back to SwiftUI. There is the same example:

.. code-block:: swift

  struct UserInfo: View {
      let users: [User]

      var body: some View {
          Group {
              if users.isEmpty {
                  Text("No users")
              } else {
                  VStack {
                      Text("Great!")
                      Text("We have \(users.count) users!")
                  }
              }
          }
      }
  }

And this is the actual code represented by it:

.. code-block:: swift

  struct UserInfo: View {
      let users: [User]

      var body: some View {
          let v: _ConditionalContent<Text, VStack<TupleView<(Text, Text)>>>
          if users.isEmpty {
              v = ViewBuilder.buildEither(first: Text("No users"))
          } else {
              v = ViewBuilder.buildEither(second: VStack {
                  return ViewBuilder.buildBlock(
                      Text("Great!"),
                      Text("We have \(users.count) users!")
                  )
              })
          }
          return v
      }
  }

Voila! All the dynamic structures are replaced by ``ViewBuilder`` method calls. In
this way, we can use a complex type to represent the structure. Like ``if``
statement will be transformed to ``ViewBuilder.buildEither`` call, and its return
value contains the information of both ``if`` block and ``else`` block.

``ViewBuilder.buildBlock`` is used to represent a child element that contains
multiple views.

With function builders, you can even create your own DSLs. And this year in WWDC20,
Apple released more features based on function builders, like **WidgetKit** and
SwiftUI **App Structure**.

How SwiftUI determine when to update a view?
--------------------------------------------
All views in SwiftUI are like **PureComponent** in React by default. That means,
all the member variables (props) will be used to evaluate the equality, of course
it's shallow comparison.

What if you want to customize the update strategy? If you take a look at the
declaration of ``View`` protocol, you will notice this subtle thing:

.. code-block:: swift

  extension View where Self : Equatable {

      /// Prevents the view from updating its child view when its new value is the
      /// same as its old value.
      @inlinable public func equatable() -> EquatableView<Self>
  }

SwiftUI provides an ``EquatableView`` to let you achieve that. All you need to do
is make your view type conform ``Equatable`` and implement the ``==`` function.
Then wrap it into ``EquatableView`` at the call-site.

.. References:

.. _`Thinking in React Hooks`: https://wattenberger.com/blog/react-hooks
.. _`[SE-0258] Property Wrappers`: https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md
.. _`Function builders (draft proposal)`: https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md
