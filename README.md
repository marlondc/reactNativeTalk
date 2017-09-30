# React Native and Redux

## Intro
- This talk/doc will give and intro into React Native with Redux.

- The example is using the `Manager` app from the Stephen Grider course on Udemy [The Complete React Native and Redux Course](https://www.udemy.com/the-complete-react-native-and-redux-course/) as a guide.

- The code is available [here](https://github.com/StephenGrider/ReactNativeReduxCasts/tree/master/manager).

- At the end of this you should know as much if not more than me, and realise how similar react and react native is.

## Setup

This example makes use of the react-native cli.
1. `$ brew install node && brew install watchman`
2. `$ npm install -g react-native-cli`
3. `$ react-native init 'app_name'`
4. `$ cd app_name && react-native run-ios`

You should then see the default Welcome to React Native screen.

## Coding

This example uses redux, react-native-router-flux, and firebase as the main dependencies. As you are pretty much writing react you can still use airbnb's linter but some rules need ~~*tweaking*~~, mainly the jsx naming convention.

This seems to be what I do for each react native project I start.

It is easiest if I show you each page and explain what each bit is doing.

### Entry points for React Native

Replace everything in `index.ios.js` and `index.android.js` with
```
import {
  AppRegistry
} from 'react-native';
import App from './src/App';

AppRegistry.registerComponent('manager', () => App);
```

This is very similar to what `ReactDOM.render(document.getElementById('app'), App)`

### App.js file

Make an App.js file in a new folder src at the root of the project

```
import React, { Component } from 'react';
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import firebase from 'firebase';
import ReduxThunk from 'redux-thunk';
import reducers from './reducers';
import Router from './Router';
import {
  FIREBASE_API_KEY,
  FIREBASE_AUTH_DOMAIN,
  FIREBASE_DATABASE_URL,
  FIREBASE_PROJECT_ID,
  FIREBASE_STORAGE_BUCKET,
  FIREBASE_MESSAGING_SENDER_ID,
} from 'react-native-dotenv';

class App extends Component {
  componentWillMount() {
    const config = {
      apiKey: FIREBASE_API_KEY,
      authDomain: FIREBASE_AUTH_DOMAIN,
      databaseURL: FIREBASE_PROJECT_ID,
      storageBucket: FIREBASE_STORAGE_BUCKET,
      messagingSenderId: FIREBASE_MESSAGING_SENDER_ID
    };

    firebase.initializeApp(config);
  }

  render() {
    const store = createStore(reducers, {}, applyMiddleware(ReduxThunk));

    return (
      <Provider store={store}>
        <Router />
      </Provider>
    );
  }
}

export default App;
```
Again this should look familiar. This is the setup for a redux, router, env variables and firebase.

Expanding on firebase it is very simple. You go to [firebase](http://firebase.google.com/), create an account and create a new project. You want to get the api secrets from the web api console option and then format it as shown above.

- Firebase is used here as database and for user authorization. I am sure that if your app is hitting an api then your api should handle the data and use that database instead. For this example it is just using the email authentication option


### Router File

```
import React from 'react';
import { Scene, Router, Actions } from 'react-native-router-flux';
import LoginForm from './components/LoginForm';
import EmployeeList from './components/EmployeeList';
import EmployeeCreate from './components/EmployeeCreate';
import EmployeeEdit from './components/EmployeeEdit';

const RouterComponent = () => {
  return (
    <Router sceneStyle={{ paddingTop: 65 }}>
      <Scene key="auth">
        <Scene key="login" component={LoginForm} title="Please Login" />
      </Scene>

      <Scene key="main">
        <Scene
          onRight={() => Actions.employeeCreate()}
          rightTitle="Add"
          key="employeeList"
          component={EmployeeList}
          title="Employees"
          initial
        />
        <Scene key="employeeCreate" component={EmployeeCreate} title="Create Employee" />
        <Scene key="employeeEdit" component={EmployeeEdit} title="Edit Employee" />
      </Scene>
    </Router>
  );
};

export default RouterComponent;
```
This is the router file, again should look familiar.
So a little terminology, routes are called scenes/scenes are called routes, whatever your outlook on life is.
`react-native-router-flux` has many options/props the docs are [here](https://github.com/aksonov/react-native-router-flux).

sceneStyle is a react native prop, that applies the styles to your scenes or components. Unlike the web there are no stylesheets, everything is like this inline styling and flexbox positioning.

`key` props is the name of the route, which allows for simple routing. react-native-router-flux has a method called Action which acts like a redirect within the app.
That explains what `onRight` is doing. It creates a part in the top of the `header` bar with a title of `rightTitle` that when pressed will change the sceen to employeeCreate

`title` is the title in the middle of the header.

`initial` will make that the scene the initial scene on load.

`component`, again should look familiar ;P that will be the component that is rendered under the header

### Reducers

```
// reducers/index.js

import { combineReducers } from 'redux';
import AuthReducer from './AuthReducer';
import EmployeeFormReducer from './EmployeeFormReducer';
import EmployeeReducer from './EmployeeReducer';

export default combineReducers({
  auth: AuthReducer,
  employeeForm: EmployeeFormReducer,
  employees: EmployeeReducer
});
```

```
reducers/AuthReducer

import {
  EMAIL_CHANGED,
  PASSWORD_CHANGED,
  LOGIN_USER_SUCCESS,
  LOGIN_USER_FAIL,
  LOGIN_USER
} from '../actions/types';

const INITIAL_STATE = {
  email: '',
  password: '',
  user: null,
  error: '',
  loading: false
};

export default (state = INITIAL_STATE, action) => {
  switch (action.type) {
    case EMAIL_CHANGED:
      return { ...state, email: action.payload };
    case PASSWORD_CHANGED:
      return { ...state, password: action.payload };
    case LOGIN_USER:
      return { ...state, loading: true, error: '' };
    case LOGIN_USER_SUCCESS:
      return { ...state, ...INITIAL_STATE, user: action.payload };
    case LOGIN_USER_FAIL:
      return { ...state, error: 'Authentication Failed.', password: '', loading: false };
    default:
      return state;
  }
};
```

Again should look familiar and if it doesn't then you're in the wrong place

### Actions

Now Stephen Grider has a slightly different pattern here, instead of declaring and exporting the action types in the action file you make a types file and do it there. Not sure how many people do it, but it's not the way Dan does it

```
// actions/types.js

export const EMAIL_CHANGED = 'email_changed';
export const PASSWORD_CHANGED = 'password_changed';
export const LOGIN_USER_SUCCESS = 'login_user_success';
export const LOGIN_USER_FAIL = 'login_user_fail';
export const LOGIN_USER = 'login_user';

export const EMPLOYEE_UPDATE = 'employee_update';
export const EMPLOYEE_CREATE = 'employee_create';
export const EMPLOYEES_FETCH_SUCCESS = 'employees_fetch_success';
export const EMPLOYEE_SAVE_SUCCESS = 'employee_save_success';
```

```
// actions/index.js

export * from './AuthActions';
export * from './EmployeeActions';
```

```
// actions/AuthActions.js

import firebase from 'firebase';
import { Actions } from 'react-native-router-flux';
import {
  EMAIL_CHANGED,
  PASSWORD_CHANGED,
  LOGIN_USER_SUCCESS,
  LOGIN_USER_FAIL,
  LOGIN_USER
} from './types';

export const emailChanged = (text) => {
  return {
    type: EMAIL_CHANGED,
    payload: text
  };
};

export const passwordChanged = (text) => {
  return {
    type: PASSWORD_CHANGED,
    payload: text
  };
};

export const loginUser = ({ email, password }) => {
  return (dispatch) => {
    dispatch({ type: LOGIN_USER });

    firebase.auth().signInWithEmailAndPassword(email, password)
      .then(user => loginUserSuccess(dispatch, user))
      .catch((error) => {
        console.log(error);

        firebase.auth().createUserWithEmailAndPassword(email, password)
          .then(user => loginUserSuccess(dispatch, user))
          .catch(() => loginUserFail(dispatch));
      });
  };
};

const loginUserFail = (dispatch) => {
  dispatch({ type: LOGIN_USER_FAIL });
};

const loginUserSuccess = (dispatch, user) => {
  dispatch({
    type: LOGIN_USER_SUCCESS,
    payload: user
  });

  Actions.main();
};
```
pretty normal stuff going on here

### Login component
```
import React, { Component } from 'react';
import { Text } from 'react-native';
import { connect } from 'react-redux';
import { emailChanged, passwordChanged, loginUser } from '../actions';
import { Card, CardSection, Input, Button, Spinner } from './common';

class LoginForm extends Component {
  onEmailChange(text) {
    this.props.emailChanged(text);
  }

  onPasswordChange(text) {
    this.props.passwordChanged(text);
  }

  onButtonPress() {
    const { email, password } = this.props;

    this.props.loginUser({ email, password });
  }

  renderButton() {
    if (this.props.loading) {
      return <Spinner size="large" />;
    }

    return (
      <Button onPress={this.onButtonPress.bind(this)}>
        Login
      </Button>
    );
  }

  render() {
    return (
      <Card>
        <CardSection>
          <Input
            label="Email"
            placeholder="email@gmail.com"
            onChangeText={this.onEmailChange.bind(this)}
            value={this.props.email}
          />
        </CardSection>

        <CardSection>
          <Input
            secureTextEntry
            label="Password"
            placeholder="password"
            onChangeText={this.onPasswordChange.bind(this)}
            value={this.props.password}
          />
        </CardSection>

        <Text style={styles.errorTextStyle}>
          {this.props.error}
        </Text>

        <CardSection>
          {this.renderButton()}
        </CardSection>
      </Card>
    );
  }
}

const styles = {
  errorTextStyle: {
    fontSize: 20,
    alignSelf: 'center',
    color: 'red'
  }
};

const mapStateToProps = ({ auth }) => {
  const { email, password, error, loading } = auth;

  return { email, password, error, loading };
};

export default connect(mapStateToProps, {
  emailChanged, passwordChanged, loginUser
})(LoginForm);
```

So this is the login scene. Again there is another pattern difference here. This component is acting as the container and component. I think you could easily break it out into how Dan does it.

As you see at the top there are a few components being pulled in from common. There is where are all the shared components are put.

### Common components

React native has it's own markup language similar to how you use html but this expand on them, the main ones are view and text, but as I show more components you'll see others.

```
// Card component

import React from 'react';
import { View } from 'react-native';

const Card = (props) => {
  return (
    <View style={styles.containerStyle}>
      {props.children}
    </View>
  );
};

const styles = {
  containerStyle: {
    borderWidth: 1,
    borderRadius: 2,
    borderColor: '#ddd',
    borderBottomWidth: 0,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 1,
    marginLeft: 5,
    marginRight: 5,
    marginTop: 10
  }
};

export { Card };
```

```
// Spinner component

import React from 'react';
import { View, ActivityIndicator } from 'react-native';

const Spinner = ({ size }) => {
  return (
    <View style={styles.spinnerStyle}>
      <ActivityIndicator size={size || 'large'} />
    </View>
  );
};

const styles = {
  spinnerStyle: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
};

export { Spinner };
```
as you can see this spinner will use the ActivityIndicator from react-native, this will be the default Android or IOS spinner

### Employee List component
```
import _ from 'lodash';
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { ListView } from 'react-native';
import { employeesFetch } from '../actions';
import ListItem from './ListItem';

class EmployeeList extends Component {
  componentWillMount() {
    this.props.employeesFetch();

    this.createDataSource(this.props);
  }

  componentWillReceiveProps(nextProps) {
    // nextProps are the next set of props that this component
    // will be rendered with
    // this.props is still the old set of props

    this.createDataSource(nextProps);
  }

  createDataSource({ employees }) {
    const ds = new ListView.DataSource({
      rowHasChanged: (r1, r2) => r1 !== r2
    });

    this.dataSource = ds.cloneWithRows(employees);
  }

  renderRow(employee) {
    return <ListItem employee={employee} />;
  }

  render() {
    return (
      <ListView
        enableEmptySections
        dataSource={this.dataSource}
        renderRow={this.renderRow}
      />
    );
  }
}

const mapStateToProps = state => {
  const employees = _.map(state.employees, (val, uid) => {
    return { ...val, uid };
  });

  return { employees };
};

export default connect(mapStateToProps, { employeesFetch })(EmployeeList);
```

The reason why I am showing you this is because of the `ListView` from `import { ListView } from 'react-native';`
The cool thing about ListView is that it will only render the list item when it is view and make the device aware that it is scrollable. The setup for it is shown above.

Forgive me but I haven't written tests for this.

To manual test you can just run `$ react-native run-ios`.
To port it onto your device follow these [instructions](https://facebook.github.io/react-native/docs/running-on-device.html).
It does seem a lot easier to test on an apple device than android. You need to install Java and Android Studio before testing on an android phone or emulator.

Instructions to upload and use an icon image can be found [here](https://stackoverflow.com/questions/34329715/how-to-add-icons-to-react-native-app). I found the yo generator option the easiest