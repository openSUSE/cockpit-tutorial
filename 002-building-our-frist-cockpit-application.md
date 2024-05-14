# Cockpit application development: The start

In the last part we downloaded starter kit, installed our dependencies, and opened the repo in out code editor. Lets continue from there. We'll start by opening app.jsx, we can see the a bare bones example application. An Application is the upstream jargon for plugins for cockpit. 

## Our first application
Right now it may seem daunting with a large amount of boiler plate, and support for translations via gettext. We are going to simplify this. To start with, lets get rid of everything. We're going to start from scratch and build up. To start with lets start with a simple react component. To start with we are going to import 2 libraries.

```js
import React from "react";
import cockpit from "cockpit";
```

The first line imports the React namespace allowing us to use react goodies like `React.useEffect`/`React.useState` and secondly we import cockpit, allowing us to use the cockpit api. For now we are going to look into the React side. Looking into the [developer reference](https://react.dev/reference/react/useEffect) for useEffect the description is

> `useEffect` is a React Hook that lets you synchronize a component with an external system.

While this may seem overly complicated it really is a component that is run when the page loads, and when one of its depencies changes. Its useful for us as it will handle our application logic. Now lets look at `useState`, the descripon on the [developer reference](https://react.dev/reference/react/useState) is rather simple and less confusing 

> `useState` is a React Hook that lets you add a state variable to your component

Use State is very simple, it allows us to create a state variable, normally variables in react components are not stateful. They will reset on every run. Having a State decouples the value of the variable from the render loop. Now Using these 2 components we'll continue writing our code.

```jsx
export const Application = () => {
    const [hostname, SetHostname] = React.useState("");

    React.useEffect(() => {
        /* application logic goes here */
    }, []);

    return <p>{`hostname: ${hostname}`}</p>;
};
```

Here we define an react component `Application`. `Application` defines a react component, all cockpit applications have a main component called `Application`. We define a statefull variable `hostname` and a helper to set it `SetHostname`. Finally we have a `useEffect` Which runs our application code once the page renders.Now lets add our application logic. 

```jsx
cockpit.spawn(["cat", "/etc/hostname"])
                .then(stdout => SetHostname(stdout))
                .catch(() => SetHostname("There was an error reading /etc/hostname"));
```

The logic is extremly simple. Here we are using the cockpit api to spawn the command `cat /etc/hostname`. Then we take the output and use it to set the hostname state variable. If there was an error executing the cockpit command we "handle" the error. Now our code should look like this.

```jsx
import React from "react";
import cockpit from "cockpit";

export const Application = () => {
    const [hostname, SetHostname] = React.useState("");

    React.useEffect(() => {
        cockpit.spawn(["cat", "/etc/hostname"])
                .then(stdout => SetHostname(stdout))
                .catch(() => SetHostname("There was an error reading /etc/hostname"));
    }, []);

    return <p>{`hostname: ${hostname}`}</p>;
};
```

Now lets test out our code. To start with, lets by build our application. `npm run build`. This will build our application. Next lets install the application. Here we have two options. One is to install it with root privilleges and avaliable too all users. Or we can install without root privilleges but only avaliable to our the current user.

With root privilleges:
```sh
$# make install
```

Without root privivleges:
```sh
$ make devel-install
```

I would recomend using the non-root privilleges option as it allows for faster development. After choosing your method and logining in with the appropriate user. You should see something similar to this page. 

<sub>note: if you see a no hostname check if /etc/hostname is empty, most recent versions of tumbleweed do not set /etc/hostname and leave it as an empty file</sub>

![](./images/basic_application.png)

So far we've been using jsx without any helpers. It looks extremely plain and borderline unusable. Before you continue if you'd like to learn more about react you can look into the further reading.

## Further reading:
- https://react.dev