# Building a Simple Fitness Application in React Native & Firebase

<h2>Outline</h2>
<ul>
    <li>Setup firebase</li>
    <li>Setup redux store</li>
    <li>Add drawer overview page</li>
    <li>Implementing the core</li>
</ul>

In this article I am going to work you through the process of building a fitness mobile application with React Native & Firebase.

### Installation guide

If you are new to React Native then follow the installation guide on the [React Native official documentation
](https://reactnative.dev/docs/environment-setup).

Run the command below to install your react native app.

```
npx react-native init whogetMobileApp --template react-native-template-redux-typescript
```

<strong>Note\*</strong>

<p style="color: lightblue; font-style: italic">We assume that you have Node version > 16 installed. If you have not yet installed then <a href="https://nodejs.org/en">download and install the latest version here</a>.</p>

Clear the initial template files inside the `src` directory. It should now look like the structure below.

```
src
    ├── App.tsx
    ├── components
    │
    ├── pages
    ├── store
    │   ├── hooks
    │   │   └── index.ts
    │   └── store.ts
    └── types.d.ts
```

### Setup firebase

We will need `firestore` in this application to save our wokouts and their exercises.

To initialize `firestore` in this application we first create a firebase project in the [firebase console](https://console.firebase.google.com/u/0/).

Inside the project, create a web application, copy it configuration keys and place inside `fbConfig.ts` at the root directory of your react native project.

Customize the `fbConfig.ts` file such that it looks like the one below.

```
import { initializeApp } from 'firebase/app';
import { initializeFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: '***********',
  authDomain: '*******************',
  projectId: '*********************',
  storageBucket: '******************',
  messagingSenderId: '**************',
  appId: '**************************',
  measurementId: '*******************',
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const db = initializeFirestore(app, {
  experimentalForceLongPolling: true,
});

export default db;
```

Replace the _(\*\*)_ asteriks with your the keys from the app you created on firebase console. Now the firestore configuration is available in all parts of the app. We just need to `import db` from the `fbConfig.ts` file and use.

Finally, run the command below to install firebase.

```
npm install firebase
```

This command installs firebase and makes available for use all client services.

### Setup redux store

If you used the template I provided at the beginning of this tutorial to create this application then `redux toolkit` and its dependencies have already been installed. Otherwise install `redux toolkit` with the commands below.

```
npm install @reduxjs/toolkit && npm install react-redux
```

Go to the repository for this project on github and copy the whole of the `src/store` directory and place in the `src/store` in your React native project. The advantage here is that they already prepared `store.ts` and `hooks.ts` files have perfectly typed hooks ( `useAppDispatch` and `useAppSelector` ) that helps you dispatch actions and ready data from the store respectively with less hastle.

We can now go on to creating a workout slice. This slice will handle all states that has to do with a workouts in the application.

```
// Create an initial state interface

export interface InitialState {
  workouts: Workout[];
  currentWorkout: Workout | null;
  status: 'idle' | 'inProgress' | 'paused' | 'ready';
  fetchStatus: 'idle' | 'loading' | 'failed';
}
```

Use the `InitialState` interface to create an initial state property for `workoutSlice`.

```
const initialState: InitialState = {
workouts: Workout[],
currentWorkout: Workout | null,
status: 'idle',
fetchStatus: 'idle',
}
```

Create the `workoutSlice` with the `createSlice` function imported from `redux/toolkit`.

```
const workoutSlice = createSlice({
    name: '/workouts',
    initialState,
    reducers: {}
});
```

We have created a basic workout slice with no `reducers`.

A reducer is a function intended to handle a specific action type. It is used to mutate the state.

Let's add some reducer functions to the `workoutSlice`.

```
const workoutSlice = createSlice({
        name: '/workouts',
        initialState,
        reducers: {
            setCurrentWorkout: (state, action) => {
                state.currentWorkout = state.workouts.filter(
                workout => state.workouts.indexOf(workout) === action.payload,
            )[0];
            },
            setCurrentState: (state, action) => {
                state.status = action.payload;
            },
            loadAllWorkouts: (state, action) => {
                state.workouts = action.payload;
            },
    }
});
```

In the code block above we added three `reducer functions`.

-   `setCurrentWorkout` which sets the current workout.
-   `setCurrentState` to update the current state of the workout (`paused`, `inProgress`, `idle`, `ready`)
-   `loadAllWorkouts` to fetch all workouts and store them on the state for easy access.

Lets create a reducer to fetch data from the firebase asynchronuously with the `createAsyncThunk` function.

```
export const fetchWorkoutsAsync = createAsyncThunk(
  'fetchAsync/fetchWorkouts',
  async (dispatch: AppDispatch) => {
    const docsRef = collection(db, 'workouts');
    const docsSnapshot = await getDocs(docsRef);

    const workouts: Workout[] = [];

    docsSnapshot.forEach(document => {
      const data = document.data();

      const workout = {
        title: data.title,
        description: data.description,
        exercises: data.exercises,
        duration: data.duration,
        postRestTime: data.postRestTime,
      };
      workouts.push(workout);
    });
    console.log(workouts);
    dispatch(loadAllWorkouts(workouts));
  },
);
```

The `fetchWorkoutsAsync` function gets all workouts from `firestore` and `dispatch` to the state using the `loadAllWorkouts` reducer we defined earlier in the list of reducers. This thunk function is available everywhere in our application and can be dispatched the same way as number reducer functions.

Let's update the `fetchStatus` of the `workouts` on the store using the `extraReducers` configuration property.

```
...
  extraReducers: builder => {
    builder
      .addCase(fetchWorkoutsAsync.pending, state => {
        state.fetchStatus = 'loading';
      })
      .addCase(fetchWorkoutsAsync.rejected, state => {
        state.fetchStatus = 'failed';
      })
      .addCase(fetchWorkoutsAsync.fulfilled, state => {
        state.fetchStatus = 'idle';
      });
  },
  ...
```

Let's make our reducers available in the application by destructuring and exporting from `workoutSlice.actions`.

```
export const { setCurrentWorkout, setCurrentState, loadAllWorkouts } =
  workoutSlice.actions;
```

Make the `workoutSlice` available to the store.

```
export default workoutSlice.reducer;
```

This is the final content of the `workoutSlice.ts` file 👇.

```
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import { collection, getDocs } from 'firebase/firestore';
import db from '../../../fbConfig';
import { InitialState, Workout } from '../../types';
import { AppDispatch } from '../store';

const initialState: InitialState = {
  workouts: [],
  currentWorkout: null,
  status: 'idle',
  fetchStatus: 'idle',
};
const workoutSlice = createSlice({
  name: 'workout',
  initialState,
  reducers: {
    setCurrentWorkout: (state, action) => {
      state.currentWorkout = state.workouts.filter(
        workout => state.workouts.indexOf(workout) === action.payload,
      )[0];
    },
    setCurrentState: (state, action) => {
      state.status = action.payload;
    },
    loadAllWorkouts: (state, action) => {
      state.workouts = action.payload;
    },
  },
  extraReducers: builder => {
    builder
      .addCase(fetchWorkoutsAsync.pending, state => {
        state.fetchStatus = 'loading';
      })
      .addCase(fetchWorkoutsAsync.rejected, state => {
        state.fetchStatus = 'failed';
      })
      .addCase(fetchWorkoutsAsync.fulfilled, state => {
        state.fetchStatus = 'idle';
      });
  },
});

export const fetchWorkoutsAsync = createAsyncThunk(
  'goals/fetchWorkouts',
  async (dispatch: AppDispatch) => {
    const docsRef = collection(db, 'workouts');
    const docsSnapshot = await getDocs(docsRef);

    const workouts: Workout[] = [];

    docsSnapshot.forEach(document => {
      const data = document.data();

      const workout = {
        title: data.title,
        description: data.description,
        exercises: data.exercises,
        duration: data.duration,
        postRestTime: data.postRestTime,
      };
      workouts.push(workout);
    });
    console.log(workouts);
    dispatch(loadAllWorkouts(workouts));
  },
);

export default workoutSlice.reducer;
export const { setCurrentWorkout, setCurrentState, loadAllWorkouts } =
  workoutSlice.actions;

```

Finally let's add the workSlice to the `store` in `store.ts`.

```
import { configureStore, ThunkAction, Action } from '@reduxjs/toolkit';
import workoutSlice from './slices/workoutSlice';

export const store = configureStore({
  reducer: {
    workout: workoutSlice,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export type AppThunk<ReturnType = void> = ThunkAction<
  ReturnType,
  RootState,
  unknown,
  Action<string>
>;
```

### Setup Drawer Navigation - The overview page

In section we want to build a left drawer that will display Each workout, and a brief description of what it is and the kind of exercises under it.

There are two ways to implement drawer nevigation in React Native

-   Using the native `DrawerLayoutAndroid` or
-   Using the `createDrawerNavigation` function from `React Navigation` library.

In this tutorial we will use `DrawerLayoutAndroid`.

We will make use of the following `DrawerLayoutAndroid` `props`

-   `drawerBackgroundColor` to set the background color of the drawer view.
-   `drawerWidth` to set the `width` of the drawer view.
-   `drawerPosition` to set the position of the drawer view - (`left`, `right`)
-   `renderNavigationView`: the props whose value is the the drawer container view.

In order to class the drawer manually we will use the react's `useRef` hook to reference the `DrawerLayoutAndroid` component and we can use its `closeDrawer()` method.

Let's define the navigation view

```
  const navigationView = () => {
    return (
      <View>
        <View>
          <Pressable
            style={appStyles.closeDrawerBtn}
            android_ripple={{ color: Colors.primaryDark.backgroundColor }}
            onPress={() => drawer.current?.closeDrawer()}>
            <Icon name="arrow-left" size={25} color={Colors.onPrimary.color} />
          </Pressable>
        </View>
        <Overview />
      </View>
    );
  };
```

Finally the drawer navigation setup is implemented in the `App.tsx` as follows

```
    <DrawerLayoutAndroid
      drawerBackgroundColor={Colors.primary.backgroundColor}
      ref={drawer}
      drawerWidth={320}
      renderNavigationView={navigationView}
      drawerPosition={'left'}>
      <Header drawer={drawer} />
      <StatusBar backgroundColor={Colors.primaryDark.backgroundColor} />
      <Home />
    </DrawerLayoutAndroid>
```

Check the `App.tsx` full setup.

### Implementing the core
