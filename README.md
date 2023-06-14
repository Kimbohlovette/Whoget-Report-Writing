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
- `npx react-native start` starts the application.

