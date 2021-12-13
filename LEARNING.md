# 20. React, Part II

## ==Lifecycle Methods==

```js
//index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Profile } from './Profile';
import { Directory } from './Directory';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      currentUsername: null,
    };
    this.handleChoose = this.handleChoose.bind(this);
    this.handleReturnToDirectoryClick = this.handleReturnToDirectoryClick.bind(
      this
    );
  }

  handleChoose(newUsername) {
    this.setState({ currentUsername: newUsername });
  }

  handleReturnToDirectoryClick() {
    this.setState({ currentUsername: null });
  }

  render() {
    let body;
    if (this.state.currentUsername) {
      body = (
        <Profile
          username={this.state.currentUsername}
          onChoose={this.handleChoose}
        />
      );
    } else {
      body = <Directory onChoose={this.handleChoose} />;
    }

    return (
      <div className="App">
        <header>
          <h1>PetBook</h1>

          <nav>
            {this.state.currentUsername && (
              <button onClick={this.handleReturnToDirectoryClick}>
                Return to directory
              </button>
            )}
          </nav>
        </header>

        <main>{body}</main>
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('app'));
```

```js
//Directory.js
import React from 'react';
import { Userlist } from './Userlist';

export function Directory(props) {
  return (
    <div className="Directory">
      <h2>User directory</h2>
      <Userlist
        usernames={['dog', 'cat', 'komodo']}
        onChoose={props.onChoose}
      />
    </div>
  );
}
```

```js
//Profile.js
import React from 'react';
import { fetchUserData, cancelFetch } from './dataFetcher';
import { Userlist } from './Userlist';

export class Profile extends React.Component {
  constructor(props) {
    super(props);
    this.state = { userData: null };
  }

  loadUserData() {
    this.setState({ userData: null });
    this.fetchID = fetchUserData(this.props.username, (userData) => {
      this.setState({ userData });
    });
  }
  
  componentDidMount() {
    this.loadUserData();
  }

  componentWillUnmount() {
    cancelFetch(this.fetchID);
  }
  
  componentDidUpdate(prevProps) {
    if (this.props.username !== prevProps.username) {
      this.loadUserData();
    }
  }

  render() {
    const isLoading = (this.state.userData === null);
    let name;
    let bio;
    let friends;
    let className = 'Profile';
    if (isLoading) {
      className += ' loading';
      name = 'Loading...';
      bio = 'Loading...';
      friends = [];
    } else {
      name = this.state.userData.name;
      bio = this.state.userData.bio;
      friends = this.state.userData.friends;
    }

    return (
      <div className={className}>
        <div className="profile-picture">
          {!isLoading && (
            <img src={this.state.userData.profilePictureUrl} 
            alt="" />
          )}
        </div>
        <div className="profile-body">
          <h2>{name}</h2>
          <h3>@{this.props.username}</h3>
          <p>{bio}</p>
          <h3>My friends</h3>
          <Userlist usernames={friends} onChoose={this.props.onChoose} />
        </div>
      </div>
    );
  }
}
```

```js
//Userlist.js
import React from 'react';

export class Userlist extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick(event) {
    this.props.onChoose(event.target.dataset.username);
  }

  render() {
    return (
      <ul>
        {this.props.usernames.map((username) => (
          <li key={username}>
            <button data-username={username} onClick={this.handleClick}>
              @{username}
            </button>
          </li>
        ))}
      </ul>
    );
  }
}
```



The component lifecycle has three high-level parts:

1.  *Mounting*, when the component is being initialized and put into the DOM for the first time

    *   `constructor()` only executes during the mounting phase, but `render()` executes during both the mounting and updating phase.

2.  *Updating*, when the component updates as a result of changed state or changed props

    *   Just like `componentDidMount()` is a good place for mount-phase setup,  `componentDidUpdate()` is a good place for update-phase work.

3.  *Unmounting*, when the component is being removed from the DOM

    *   *In general, when a component produces a side-effect, you should remember to clean it up.*

        JavaScript gives us the [`clearInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/clearInterval) function. `setInterval()` can return an ID, which you can then pass into `clearInterval()` to clear it. 

Every React component you’ve ever interacted with does the first step at a minimum. If a component never mounted, you’d never see it!

![image-20211212112556699](https://tva1.sinaimg.cn/large/008i3skNgy1gxbij861m4j30vu0ie409.jpg)