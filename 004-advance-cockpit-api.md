# Advanced cockpit

So far our application has grown from a just a string on the page to now a more fleshed out semi passable view. Lets go a step further and make it more useful. Currently we are reading our data from `/etc/hostname` however that isn't graunteed to have data nor is graunteed to be the actual hostname on a system using systemd. Lets decouple oursevels from files and instead using dbus and use that to get a little more information

So systemd offers a dbus api `org.freedesktop.hostname1` which returns a a nice json object with a lot of system information. Getting and displaying this will be a little more invovled then other parts. We'll be breaking this into 2 parts. In the first we'll simply change our data source to the dbus api and use the "pretty host name" instead of the `/etc/hostname`. At this point your code should look like.


```jsx
import React from "react";
import cockpit from "cockpit";

export const Application = () => {
    const [hostname, SetHostname] = React.useState("");

    React.useEffect(() => {
       cockpit.file("/etc/hostname")
            .watch((content) => {
                if (content) {
                    SetHostname(content);
                }
            });
    }, []);

    return (
        <Card>
            <CardTitle>Hostname</CardTitle>
            <CardBody>running on {hostname}</CardBody>
        </Card>
    );
};
```

Lets staty with replacing the `cockpit.file` api with some code to work with the dbus api. Before we continue I want to just state this is a extemely small glance at the dbus api. We are using the very basics here.

```jsx
const client = cockpit.dbus("org.freedesktop.hostname1");
client.call("/org/freedesktop/hostname1", "org.freedesktop.hostname1", "Describe").then(([data]) => {
                    SetHostname(JSON.parse(data).PrettyHostname);
                });
```

We start by creating a dbus client, we specify the `org.freedesktop.hostname1` interface. Next we use the `client.call` function. Call greatly simplfies our work, as we don't intened to reuse the `proxy`. This saves a large amount of work creating dbus `proxies` however this does mean we can't use interactive apis with `.call`. As `call` returns a promise we wait for the data to return.

You might have already noticed data is in an array. All data returned by `call` returns the data in an array, hence we use `[data]` to access the data instead the array without manually acessing the data. Next we use `JSON.parse(data)` to convert the returned string to json. `org.freedesktop.hostname1.Describe` returns json data. We then access the `PrettyHostname` key and set the hostname. In the end our page should look like this now

![](./images/prettyhostname_time.png)

Your code should look like this now

```jsx
import React from "react";
import cockpit from "cockpit";

export const Application = () => {
    const [hostname, SetHostname] = React.useState("");

    React.useEffect(() => {
       const client = cockpit.dbus("org.freedesktop.hostname1");
       client.call("/org/freedesktop/hostname1", "org.freedesktop.hostname1", "Describe")
       .then(([data]) => {
                    SetHostname(JSON.parse(data).PrettyHostname);
        });
    }, []);

    return (
        <Card>
            <CardTitle>Hostname</CardTitle>
            <CardBody>running on {hostname}</CardBody>
        </Card>
    );
};
```

In the next part we will be using more of this data and play a little more with patternfly.