# flutter_redux

[![build status](https://gitlab.com/brianegan/flutter_redux/badges/master/build.svg)](https://gitlab.com/brianegan/flutter_redux/commits/master)  [![coverage report](https://gitlab.com/brianegan/flutter_redux/badges/master/coverage.svg)](https://brianegan.gitlab.io/flutter_redux/coverage/)

A set of utilities that allow you to easily consume a [Redux](https://pub.dartlang.org/packages/redux) Store to build Flutter Widgets.

This package is built to work with [Redux.dart](https://pub.dartlang.org/packages/redux). If you use Greencat, check out [flutter_greencat](https://pub.dartlang.org/packages/flutter_greencat). 

## Redux Widgets 

  * `StoreProvider` - The base Widget. It will pass the given Redux Store to all descendants that request it.
  * `StoreBuilder` - A descendant Widget that gets the Store from a `StoreProvider` and passes it to a Widget `builder` function.
  * `StoreConnector` - A descendant Widget that gets the Store from the nearest `StoreProvider` ancestor, converts the `Store` into a `ViewModel` with the given `converter` function, and passes the `ViewModel` to a `builder` function. Any time the Store emits a change event, the Widget will automatically be rebuilt. No need to manage subscriptions!

## Usage

Let's demo the basic usage with the all-time favorite: A counter example!

```dart
import 'package:flutter/material.dart';
import 'package:redux/redux.dart';
import 'package:flutter_redux/flutter_redux.dart';

// Start by creating your normal "Redux Setup."

// One simple action: Increment
enum Actions { Increment }

// The reducer, which takes the previous count and increments it in response
// to an Increment action.
int counterReducer(int state, action) {
  if (action == Actions.Increment) {
    return state + 1;
  }

  return state;
}

void main() {
  runApp(new FlutterReduxApp());
}

class FlutterReduxApp extends StatelessWidget {
  // Create your store as a final variable in a base Widget. This works better
  // with Hot Reload than creating it directly in the `build` function.
  final store = new Store(counterReducer, initialState: 0);

  @override
  Widget build(BuildContext context) {
    final title = 'Flutter Redux Demo';

    return new MaterialApp(
      theme: new ThemeData.dark(),
      title: title,
      home: new StoreProvider(
        // Pass the store to the StoreProvider. Any ancestor `StoreConnector`
        // Widgets will find and use this value as the `Store`.
        store: store,
        // Our child will be a `StoreConnector` Widget. The `StoreConnector`
        // will find the `Store` from the nearest `StoreProvider` ancestor,
        // convert it into a ViewModel, and pass that ViewModel to the
        // `builder` function.
        //
        // Every time the button is tapped, an action is dispatched and run
        // through the reducer. After the reducer updates the state, the Widget
        // will be automatically rebuilt. No need to manually manage
        // subscriptions or Streams!
        child: new Scaffold(
          appBar: new AppBar(
            title: new Text(title),
          ),
          body: new Center(
            child: new Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                new Text(
                  'You have pushed the button this many times:',
                ),
                // Connect the Store to a Text Widget that renders the current
                // count.
                new StoreConnector<int, String>(
                  converter: (store) => store.state.toString(),
                  builder: (context, count) => new Text(
                        count,
                        style: Theme.of(context).textTheme.display1,
                      ),
                )
              ],
            ),
          ),
          // Connect the Store to a FloatingActionButton. In this case, we'll
          // use the Store to build a callback that with dispatch an Increment
          // Action.
          //
          // Then, we'll use this callback to the button's `onPressed` handler.
          floatingActionButton: new StoreConnector<int, VoidCallback>(
            converter: (store) {
              // Return a `VoidCallback`, which is a fancy name for a function
              // with no parameters.
              return () => store.dispatch(Actions.Increment);
            },
            builder: (context, callback) => new FloatingActionButton(
                  // Attach the onIncrementPressed VoidCallback
                  onPressed: callback,
                  tooltip: 'Increment',
                  child: new Icon(Icons.add),
                ),
          ),
        ),
      ),
    );
  }
}

```  
