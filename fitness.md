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

Visit [my github repository](https://github.com/Kimbohlovette/FitnessCoach) for the final content of the `workoutSlice.ts` file.

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

We want to jump into the real business. The countdown logic is done in the `Home.tsx` file.

Firstly pull out all the workouts from the store as follows

```
const workouts = useAppSelector(state => state.workout.workouts);
```

Define local states `workout`, `timer`, `index`, `countState` to hold the current workout, timer value, index of each workout in the workouts array and the state of the countdown respectively.

```
  const [workout, setWorkout] = useState<Workout>(workouts[0]);
  const [timer, setTimer] = useState<number>(workout.duration);
  const [index, setIndex] = useState<number>(0);
  const [countState, setCountdownState] = useState<
    'idle' | 'inProgress' | 'resting'
  >('idle');
```

Define a function `setNextWork` that sets the next workout. It takes the previous index as parameter.

```
  const setNextWorkout = (prevIndex: number) => {
    if (prevIndex >= workouts.length - 1) {
      setWorkout(workouts[0]);
      return 0;
    } else {
      setWorkout(workouts[prevIndex + 1]);
      return prevIndex + 1;
    }
  };
```

Define the `startTimerCountdown()`. This function does the countdown of the current workout. That is, the workout which is currently held in `workout` state variable.

```
  const startTimerCountdown = () => {
    setTimer(workout.duration);
    setCountdownState('inProgress');
    const countId = setInterval(() => {
      setTimer(state => {
        if (state === 0) {
          clearInterval(countId);
          setCountdownState('resting');
          return 0;
        }
        if (state <= 10) {
          RNSystemSounds.beep();
        }
        return state - 1;
      });
    }, 1000);
  };
```

We define the `startRestTimerCountown()`. This function counts down on the duration of the rest of the current workout.

```
  const startRestTimerCountdown = () => {
    setTimer(workout.postRestTime + 5);
    const restCountId = setInterval(() => {
      RNSystemSounds.beep();
      setTimer(state => {
        if (state === 0) {
          setCountdownState('idle');
          clearInterval(restCountId);
          setIndex(setNextWorkout(index));
          return 0;
        }
        return state - 1;
      });
    }, 1000);
  };
```

The last function is the `startCoundown()` function which is what triggers the other functions. It is used in the `onPress()` callback to trigger start when the button is clicked.

```
  const handleStart = () => {
    startTimerCountdown();
  };
```

We want to start counting immediately the `index` value changes. This is because the the next index value is set only at the end of the current workout. So it can also be used as a signal that it's time to start another countdown.

```
  useEffect(() => {
    Tts.speak(workout.title);
    if (index) {
      startTimerCountdown();
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [index]);
```

We want to start counting rest time when the state of the `countState` state variable changes to `resting`.

```
  useEffect(() => {
    if (countState === 'resting') {
      Tts.speak('Break Time');
      startRestTimerCountdown();
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [countState]);
```

Visit [my github repository](https://github.com/Kimbohlovette/FitnessCoach) for a complete `Home.tsx` file

-   All styles are objects are imported from the `homeStyles.ts` file.

-   As a side note, I want to indicate that all icons used in this exercise were imported from `react-native-vector-icons` library. So If you are following, you can download as follows

```
npm install --save react-native-vector-icons
```

-   In some parts of the code in this `Home.tsx` I used `Tts.speak()`. This is a text to speech react native library that reads every string you pass to it at loud in English.

```
npm install --save react-native-tts
```

Congratulations! You have successfully implemented a fitness mobile application. The debug version should be good for now.

If you want to generate a `release` `apk` or `release` `aab` so that you can publish on playstore, follow the [Publishing to play store](https://reactnative.dev/docs/signed-apk-android) section of the react native documentation.

You can get the complete project on [my github repository](https://github.com/Kimbohlovette/FitnessCoach).
