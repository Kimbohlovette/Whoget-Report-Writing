# How I built the whoget full stack application from start to finish
`WhoGet` is a mobile app that connects people with certain needs directly with those who can provide these needs. The frontend for this application is made up of a mobile application for users manage their asks and the web interface where the administrators can login and monitor and manage the smooth running of application. The backend of this application was built with `Express` - a `NodeJS framework` for building web API's and `MongoDB Atlas database`.

When this application's brief was given, I went through it and generated a Software Requirement Specification document `SRS`, submitted for validation. [You can take a look at the software requirement specification here](https://docs.google.com/document/d/1Gqdp4_A4ataLCGjzyRCY55fbUDIFN9iUwZNBSY8NGE8)

When the `SRS` was validated the development process began.

## Frontend Mobile application
Instructions to setup reactive native application can be found on [React native official documentation](https://reactnative.dev/docs/environment-setup).
When you have successfully setup your environment go ahead and create a react native application with the command below
```
npx react-native init whogetMobileApp --template react-native-template-redux-typescript
```
The command above creates a react native app with typescript, redux together with required dependencies put together and configured.
### Typescript
TypeScript is a strongly typed programming language that builds on JavaScript, giving you better tooling at any scale. For more information about typescript, [visit typescript official docs here](https://www.typescriptlang.org/docs/).

### Redux
Redux is a javascript library for application state management. It will be used to manage the state data in our application. [Learn more about redux here](https://redux.js.org/introduction/getting-started).

Other packages used in this app worth mentioning are: `react-native-vector-icons` for high quality svg icons rendered as components, `Nativewind` - a tailwindcss version for react native used for styling,etc. For a list of all dependencies checkout the `package.json` on my github repo that will be presented subsequently.

### Debugging
In order to debug your react native application you will need to either setup an Android device simulator on Android studio or use USB debugging method. See instructions on setting up your app for debugging.
1. [Android Simulator](https://developer.android.com/studio/debug)
2. [USB debugging](https://developer.android.com/studio/run/device#connect)

### Start application
After you have successfully setup your debugging method run the command below to start your application.
```
npx react-native run-android && npx react-native start
```

- `npx react-native run-android` command builds the application while 
- `npx react-native start` starts the application

Clear all starter code in `App.tsx`, delete the features folder that was created for `redux slices` and clear all starter style files in the application.

#### Main pages for application
These are the main pages that make the application
1. `Asks` - The page that displays all the asks or posts that people publish on the app.
2. `AskDetail` - The page that displays the details and various actions that one can make on an ask.
3. `CreateAsk` - Form for creating new posts or asks.
4. `Signin/Signup` - The pages to authenticate users of the application.
5. `UserProfile` - Page to display details about an authenticated user.
6. `Profile` - To display current user profile.

Other pages include, `Notifications` page, `AdditionalSignupInfo` page, `Search` page etc. which are also useful in this application. But we will not use them in this article most often. You can find all of these pages and its contents in the github repository.

### Setting up navigation between pages
There are two types of navigation we need to implement. `Tap Navation` and `Stack navigation`
Firstly we define the types for these pages
``` 
import { CompositeNavigationProp } from '@react-navigation/native';
import { NativeStackScreenProps } from '@react-navigation/native-stack';

export type HomeStackParamList = {
  Asks: undefined;
  CreateAsk: { mode: 'create' | 'edit'; askId?: string };
  Authentication?: CompositeNavigationProp<AuthStackParamList>;
  UserDetails: { userId: string };
  AskDetails: { askId: string };
  Search: undefined;
};

export type RootTabParamList = {
  Home: CompositeNavigationProp<HomeStackParamList>;
  Profile: undefined;
  Notifications: undefined;
};

export type AuthStackParamList = {
  Signin: undefined;
  Signup: undefined;
  Categories: undefined;
  AdditionalSignupInfo: undefined;
};
```
On the code block above we have 
- The `HomeStackParamList` type whose properties are the various pages with their navigation parameter types. That's `Asks`, `CreateAsks`, `Authentication` which is itself made up of other pages (`Signin`, `Signup`, `Categories`, etc).
- The `AuthStackParamsList` type which contains  `Signin`, `Signup`, `Categories`, etc pages and their navigation parameter types.
- Finally we have the `RootTabParamsList`. This is the tab navigation params list which contains tab items that are visible at the bottom tab. That's  the `Home`, `Profile` and `Notifications` pages.
In other to install and setup react navigation visit [React Native official docs here](https://reactnative.dev/docs/navigation) and the [React navigation library here](https://reactnavigation.org/docs/getting-started/)
The navigations (Stack and Tab) are implemented as follow.

In the `App.tsx` component these navigations are implemented with the types defined above.

The home screen navigations
```
const HomeScreen = () => {
  return (
    <>
      <Stack.Navigator>
        <Stack.Group
          screenOptions={{
            header: NavHeader,
            headerTintColor: 'white',
            headerStyle: { ...Styles.bgPrimary },
          }}>
          <Stack.Screen name="Asks" component={Asks} />
          <Stack.Screen
            name="CreateAsk"
            component={CreateAsk}
            options={{ headerTitle: 'Create ask' }}
          />
          <Stack.Screen
            name="AskDetails"
            component={AskDetails}
            options={{ title: 'Ask detail' }}
          />
          <Stack.Screen
            name="UserDetails"
            component={UserDetails}
            options={{ title: 'User details' }}
          />
          <Stack.Screen name="Authentication" component={AuthScreen} />
          <Stack.Screen name="Search" component={Search} />
        </Stack.Group>
      </Stack.Navigator>
    </>
  );
};
```

The Authentication navigation flow.
```
const AuthStack = createNativeStackNavigator<AuthStackParamList>();
const AuthScreen = () => {
  return (
    <AuthStack.Navigator screenOptions={{ headerShown: false }}>
      <AuthStack.Screen name="Signup" component={Signup} />
      <AuthStack.Screen name="Signin" component={SignIn} />
      <AuthStack.Screen
        name="AdditionalSignupInfo"
        component={AdditionalSignupInfo}
      />
      <AuthStack.Screen name="Categories" component={UserPreferences} />
    </AuthStack.Navigator>
  );
};
```

All put together based on the `isAuthenticated` state of the user can be seen in the `App.tsx` component on github repo.

### Setting up redux store.

States useful to our application are states about current user, categories, cities or places. It makes sense to save this information in states for easy access when our application starts because they are not likely to change and the user uses them more often. I will demonstrate setup for user `userSlice` information here and the rest of the slices follow the same pattern.
```
import { PayloadAction, createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import { fetchOneUserById } from '../../apiService/fetchingFunctions';
export interface InitialState {
  isAuthenticated: boolean;
  user: any;
  status: 'loading' | 'idle' | 'failed' | 'successful';
}
const initialState: InitialState = {
  isAuthenticated: false,
  user: null,
  status: 'idle',
};
const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    updateProfile: (state, action: PayloadAction<any>) => {
      state.user = action.payload;
    },
    updateAuthStatus: (state, action: PayloadAction<boolean>) => {
      state.isAuthenticated = action.payload;
    },
  },
  extraReducers: builder => {
    builder
      .addCase(fetchUserById.fulfilled, state => {
        state.status = 'idle';
      })
      .addCase(fetchUserById.pending, state => {
        state.status = 'loading';
      })
      .addCase(fetchUserById.rejected, state => {
        state.status = 'failed';
      });
  },
});

export const { updateProfile, updateAuthStatus } = userSlice.actions;

export default userSlice.reducer;

export const fetchUserById = createAsyncThunk(
  'users/user',
  async (userId: string, { dispatch }) => {
    fetchOneUserById(userId)
      .then(data => {
        dispatch(updateProfile(data));
      })
      .catch(error => {
        console.log('An error occured in fetch api: ', error);
        dispatch(updateProfile(null));
      });
  },
);
```

In the code block above I started by creating an interface `InitialState` which defines the types of all properties that our user state contains.
- `isAuthenticated` of type `boolean` (`true` or `false`) is the property of the user start which holds the information as to whether the current user is authenticated or not.
- `user` is the property whose value is the information about the current user (`email`, `phoneNumber`, `location`, etc) that are used to populate the profile and also to use them to perform actions like creating a post, updating user profile etc.
- `status` is a property used by async reducers to determine the state of fetching in the slice.


```
  export interface InitialState {
  isAuthenticated: boolean;
  user: any;
  status: 'loading' | 'idle' | 'failed' | 'successful';
}
```

Next, week initialize the state as follows
```
const initialState: InitialState = {
  isAuthenticated: false,
  user: null,
  status: 'idle',
};
```
The next thing is to create the user slice with the `createSlice` function exported from redux toolkit. Creating a slice takes an object literal with the following properties `name`,  `initialState`, `reducers`, `extraReducers` etc. But we are going to talk only about the ones listed above.
.
- the `name` property takes a unique string which is used by redux toolkit under the hood with the name of each reducer function to create it type.
- `initialState` property which is the initial value of the user state object as defined earlier.
- `reducers`: These are the functions that will be used to dispatch actions that mutates the state of this slice directly although under the hood it don't `mutate` the state, `redux toolkit` makes it seem true.
- `extraReducers`: These are functions that can tap into asynchronuous operations like `asyncThunk` functions or middlewares created with the redux toolkit's `createAsyncThunk` function to update state base on states of the asynchronuous functions operations.[ Learn more about createAsyncThunk](https://red ux-toolkit.js.org/api/createAsyncThunk).

After creating the `userSlice` the next thing is to export all the reducers actions
```
export const { updateProfile, updateAuthStatus } = userSlice.actions;
```
Notice that the middleware function `fetchUserById` is already exported when it was defined so it is already available to other parts of the app and can be used in dispatch to dispatch state actions just like normal reducers.

Finally we export  our slice as `userSlice.reducer`
```
export default userSlice.reducer;
```
To make this slice available to redux store, open the `src/store/store.ts` add the user reducer
```
import { configureStore, ThunkAction, Action } from '@reduxjs/toolkit';
...
import userReducer from './slices/userSlice';
export const store = configureStore({
  reducer: {
    user: userReducer,
  ...
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
The full userSlice is found in `/src/store/slices/userSlice.ts` [Github repo](https://github.com/Kimbohlovette/whoget-mobile-client.git).

### Data Fetching
All functions for fetching data are found in an `src/apiService/fetchingFunctions.ts` file and exported so that they can be available in any part of the application for use. Data is fetched from an already created backend api server with base url `https://whoget-app-server.onrender.com/api/v1/`. [The full api documentation can be found here](https://github.com/Kimbohlovette/whoget-app-server). 

![image](https://github.com/Kimbohlovette/Whoget-Report-Writing/assets/37558983/7ffd655e-257e-41dc-8600-5635fb73f1b4)

These fetching functions are then used in various pages as needed. 

Example of data fetching in the `Asks.tsx` page.

![image](https://github.com/Kimbohlovette/Whoget-Report-Writing/assets/37558983/08b60ad9-58fd-420f-8421-6f517b53ddbf)

The above image demonstrates usage of `fetchPaginatedAsks()` fetching function when a user refreshes a page by pulling down the screen. This is a basic way in which data is pulled and used from the backend. They other pages and components follow this same pattern for data fetching.


### Firebase Services 
In this application we used `firebase storage` for storing images of posts or asks and firebase's `Google Authentication` service.
To setup firebase, follow the [React Native firebase documentation here](https://rnfirebase.io/).

After setting up firebase as recommended above, we can use it to store images as seen below in `src/pages/createAsk.tsx` file.
![Screenshot_3](https://github.com/Kimbohlovette/Whoget-Report-Writing/assets/37558983/199e0fd1-853a-4065-af98-a0df21198974)

Also open successful setup of `Google authentication` in app. We import the `signinWithGoogle` function from `firebase/auth` and use it as below in `src/pages/Signup.tsx`
![Screenshot_4](https://github.com/Kimbohlovette/Whoget-Report-Writing/assets/37558983/bafb06af-4b4f-4bee-a22f-190eba3960a0)

Did you notice how we also used redux toolkit to `dispatch` the `updateProfile` action we saw in `userSlice.ts` earlier? 
What about the `navigation.navigate('AdditionalSignupInfo')`? 😊

### Publishing your application
After building your application, you can generate an `apk` that can be installed in your application. To do this navigate to the root project directory and then to the `/android` directory. Run the command below.
```
./gradlew assembleRelease
```
This command builds your applicaton and create an apk which can be found in `./android/app/build/outputs/apk/release/` directory
For more instructions on how to build a signed apk [visit the React Native official documentation](https://reactnative.dev/docs/signed-apk-android)

To generate the AAB file for publishing to playstore, run the command below.

```
./gradlew bundleRelease
```
This command generates an release.aab file which is found in `android/app/release/` directory. 
- This `release.aab` file is now ready to published to playstore.
- On the google play console create a new app and navigate to its `Dashboard`. 
- On the side menu tab on `Testing> Internal testing` and upload the `release.aab` file. 
- Add the `emails` of all your `testers` by navigating to the `Testers` tab.
- Scroll to the end and copy `link` to your test app on playstore and share with your testers.
- When all tests are done and all reviews fixed, publish app on `Open testing` and urge even more people to download the app on playstore and drop reviews.
- Publish in `Production` and that's it. You app is available for production.🎉

NOTE* _In order for firebase services to work in your app published on playstore you need to navigate to `Setup> App Integrity`, click on `App signing` tab and copy the `SHA-1` string and add to the list of `Fingerprints` on the firebase app you created for your react native Project._

## Backend API
We will see how to build the backend `api` endpoints to query and manipulate data in the database. We will use `ExpressJS`, `MongoDB`, `Mongoose`, `firebase-admin`.

- `Express` is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications. [Learn more about Express here.](https://expressjs.com/).
- `Mongoose` is a Object Document Mapper for MongoDB and NodJS. Mongoose provides a straight-forward, schema-based solution to model your application data. [Learn more about mongoose here](https://mongoosejs.com/).
- `firebase-admin` is a set of server libraries that lets you interact with Firebase from privileged environments to perform actions like create your own simplified admin console to do things like look up user data or change a user's email address for authentication. [Learn more about firebase-admin here](https://firebase.google.com/docs/admin/setup).
- `MongoDB` is a non relational database management system for storing and managing a wide range of data types.

### Codebase structure

```
src
│   ├── app.ts
│   ├── controllers
│   │   ├── ask
│   │   │   ├── createAsk.ts
│   │   │   ├── deleteAsk.ts
│   │   │   ├── getAllAsks.ts
│   │   │   ├── getAskById.ts
│   │   │   ├── getPagenatedAsks.ts
│   │   │   ├── updateAskStatus.ts
│   │   │   └── updateAsk.ts
│   │   ├── category
│   │   │   ├── createCategory.ts
│   │   │   ├── deleteCategory.ts
│   │   │   ├── getAsksByCategory.ts
│   │   │   └── getPaginatedCategories.ts
│   │   ├── searchAndFilters
│   │   │   ├── filterAsks.ts
│   │   │   └── search.ts
│   │   └── user
│   │       ├── createUser.ts
│   │       ├── deleteUser.ts
│   │       ├── getAllUsers.ts
│   │       ├── getAsksByUserId.ts
│   │       ├── getPaginatedUsers.ts
│   │       ├── getUserByEmail.ts
│   │       ├── getUserById.ts
│   │       ├── getUserByUid.ts
│   │       ├── updateUserStatus.ts
│   │       └── updateUser.ts
│   ├── firebase.ts
│   ├── middlewares
│   │   └── auth.ts
│   ├── mockdata.json
│   ├── models
│   │   ├── askModel.ts
│   │   ├── categoryModel.ts
│   │   └── userModel.ts
│   ├── populateDatabase.js
│   ├── routes
│   │   ├── askRoutes.ts
│   │   ├── categoryRoutes.ts
│   │   ├── searchRoutes.ts
│   │   └── userRoutes.ts
│   └── types.d.ts
└── tsconfig.json

```
We start by initializing our app inside `app.ts`

```
const app = express(); // express is imported from express
```
Connect our app to mongodb using a mongodb url copied from mongodb app created at `https://mongodb.com`

```
mongoose
	.connect(mongoConnectionPath)
	.then(() => {
		app.listen(port, () => {
			console.log(`Listening on port ${port}...`);
		});
	})
	.catch((error) => {
		console.log(error);
	});
```
We then add some `express middlewares` to app.

```
app.use(express.static('./public'));
app.use(express.json());
```
Before you start the dev server, install `nodemon` with `npm install --dev nodemon` and head to the `package.json` file and add `"dev": "nodemon ./src/app.ts",` to the `script` section.

Start the dev server with `npm run dev`. You minimal app should be up and running by now. We head over to creating models for our data.

### Creating data Models

The whoget application has three main models namely: `Users`, `Asks`, `Categories`. We are going to demonstrate on the `User` model and the rest will follow.

To create the `User model` we first need to create a `UserSchema` and use it to create a model.

```
import mongoose from 'mongoose';

const UserSchema = new mongoose.Schema(
	{
		name: {
			type: String,
			required: [true, 'Name field cannot be empty.'],
		},
		phoneNumber: {
			type: String,
			required: false,
		},
		email: {
			type: String,
			required: [true, 'Email field cannot be empty.'],
		},
		location: {
			type: String,
			required: false,
		},
		profileImage: {
			type: String,
			required: false,
		},
		role: {
			type: String,
			required: [true, 'Profile Image Url field cannot be empty.'],
		},
		status: {
			type: String,
			required: [true, 'User status field cannot be empty.'],
		},
		activities: {
			type: String,
			required: false,
		},
		uid: {
			type: String,
			required: [true, 'uid field is required'],
		},
	},
	{ timestamps: true }
);
UserSchema.index({ '$**': 'text' });
const User = mongoose.model('User', UserSchema);

export { User };
```
On the code block above, we created a `UserSchema` and used it to create and export the `User` model. The `mongoose.Schema()` takes an object parameter whose `(key,value)` pairs denote the properties of the `User` and their corresponding values denote the validation rules for that property or fields.

This line of code `const User = mongoose.model('User', UserSchema);` creates the `User` model.

### Controllers
All controllers for each route a written inside the `controllers/` directory. Controllers are responsible for handling incoming requests and returning responses to the client. We will only use them to create routes.


