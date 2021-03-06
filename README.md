# NGRX Actions

Actions/reducer utility for NGRX. It provides 4 functions to make NGRX/redux more Angular-tastic.

- `@Store(MyInitialState)`: Decorator for default state of a store.
- `@Action(...MyActionClass: Action[])`: Decorator for a action function.
- `ofAction(MyActionClass)`: Lettable operator for NGRX Effects
- `createReducer(MyStoreClass)`: Reducer bootstrap function

Inspired by [redux-act](https://github.com/pauldijou/redux-act) and [redux-actions](https://github.com/reduxactions/redux-actions) for Redux.

See [changelog](CHANGELOG.md) for latest changes.

## Whats this for?
This is _sugar_ to help reduce boilerplate when using Redux patterns. That said, here's the high level of what it provides:

- Reducers become classes so its more logical organization
- Automatically creates new instances so you don't have to handle spreads everywhere
- Enables better type checking inside your actions
- Reduces having to pass type constants by using type checking

Its dead simple (<100LOC) and you can pick and choose where you want to use it.

## Getting Started
To get started, lets install the package thru npm:

```
npm i ngrx-actions --S
```

Next, create an action just like you do with NGRX today:

```javascript
export class MyAction implements Action {
   readonly type = 'My Action';
   constructor(public payload: MyObj) {}
}
```

then you create a class a decorate it with a `Store` decorator that contains
the initial state for your reducer. Within that class you define methods
decorated with the `Action` decorator with an argument of the action class
you want to match it on.

```javascript
import { Store, Action } from 'ngrx-actions';

@Store({
    collection: [],
    selections: [],
    loading: false
})
export class MyStore {
    @Action(Load, Refresh)
    load(state: MyState, action: Load) {
        state.loading = true;
    }

    @Action(LoadSuccess)
    loadSuccess(state: MyState, action: LoadSuccess) {
        state.collection = [...action.payload];
    }

    @Action(Selection)
    selection(state: MyState, action: Selection) {
        state.collection = [...action.payload];
    }

    @Action(DeleteSuccess)
    deleteSuccess(state: MyState, action: DeleteSuccess) {
        const idx = state.collection.findIndex(r => r.myId === action.payload);
        const collection = [...state.collection];
        collection.splice(idx, 1);
        return { ...state, collection };
    }
}
```

You may notice, I don't return the state. Thats because if it doesn't see
a state returned from the action it inspects whether the state was an
object or array and automatically creates a new instance for you. If you are
mutating deeply nested properties, you still need to deal with those yourself.

You can still return the state yourself and it won't mess with it. This is helpful
for if the state didn't change or you have some complex logic going on. This can be
seen in the `deleteSuccess` action.

Above you may notice, the first action has multiple action classes. Thats because
the `@Action` decorator can accept single or multiple actions.

To hook it up to NGRX, all you have to do is call `createReducer` function passing
your store. Now pass the `myReducer` just like you would a function with a switch statement inside.

```javascript
import { createReducer } from 'ngrx-actions';
export const myReducer = createReducer(MyStore);
```

then pass that to your NGRX module just like normal:

```javascript
@NgModule({
   imports: [
      StoreModule.forRoot({
         myReducer
      })
   ]
})
export class AppModule {}
```

If you want to use NGRX effects, I've created a lettable operator that will allow you to
pass the action class as the argument like this:

```javascript
import { ofAction } from 'ngrx-actions';

@Injectable()
export class MyEffects {
    constructor(
        private update$: Actions,
        private myService: MyService
    ) {}

    @Effect()
    Load$ = this.update$.pipe(
        ofAction(Load),
        switchMap(() => this.myService.getAll()),
        map(res => new LoadSuccess(res))
    );
}
```

## Common Questions
- _What about composition?_ Well since it creates a normal reducer function, you can still use all the same composition fns you already use.
- _Will this work with normal Redux?_ While its designed for Angular and NGRX it would work perfectly fine for normal Redux. If that gets requested, I'll be happy to add better support too.
- _Do I have to rewrite my entire app to use this?_ No, you can use this in combination with the tranditional switch statements or whatever you are currently doing.