
<p align="center">

<img src="https://github.com/homebridge/branding/raw/master/logos/homebridge-wordmark-logo-vertical.png" width="200">
<br />
<img src="https://www.cnet.com/a/img/resize/148e264de9d8cc9b7a97bbe2459402fda2406809/hub/2021/01/06/19ab53c1-6b8d-4f44-a89a-b84fc7f825e8/ogi-kia.jpg?auto=webp&fit=crop&height=675&width=1200" width="200">

</p>


# KIA Connect

This plug-in uses an undocumented KIA API to start, stop and lock your KIA. This API was discovered by insepecting the network traffic in the KIA Connect web app.

This plug-in will create a **switch** to turn the engine / climate control on and off as well as a **lock mechanism** to lock / unlock the doors. Several sensors will also be created so that you can build automations:
1. Contact sensors for front left, front right, back left, back right doors as well as the hood and trunk.
2. Exterior temperature sensor
3. Battery level with low battery status
4. Occupancy sensor that is "occupied" while the engine is running

> Frequently calling this API could result in the consumption of your car's battery power. By default, we make one request per hour and make up to 5 requests after requesting a change of state. This is similar to how the KIA web app works.

### Future Work
- [x] Battery percentage
- [ ] Fuel percentage
- [x] Exterior weather
- [ ] Heated / air conditioned seats
- [ ] Defrost
- [ ] Heated steering wheel
- [x] Occupancy sensor when engine is on
- [ ] Add support for other countries (Canada, Europe)

### Known Issues
When flipping a switch in HomeKit, it could be several seconds before the desired state is achieved. This is due to the aysnc nature and overall speed of KIA's APIs. Typically, a remote command takes around 6 seconds to send, then around 10-15 seconds to be applied to the car.

The door contact sensors all have the same name. Unfortunately this is a limitation of a HomeKit. To overcome this, I would need to refactor the code to create a new Accessory per sensor.

## Installation instructions
You will need the following items:
1. Your username / password for the [KIA Connect](https://owners.kia.com/us/en/about-uvo-link.html) web app.
2. Your VIN number

### Target Temperature
When the car is started, we will attempt to bring the cabin to the specified temperature. By default, your car will run for 5 minutes, then turn off. This value should be given in Farenheit.

### Refresh Interval
When the plugin is first started, we get your car's latest info from KIA. After that, we will refresh that info every `refreshInterval` until the plugin is stopped. This value should be given in milliseconds.

In addition to refreshing every `refreshInterval` we will also request vehicle info after an action is requested. This helps HomeKit stay in sync since these actions take some time and could fail due to factors outside of our control.

### Sample config
```json
{
    "platforms": [
        {
	   "name": "homebridge-kia-connect",
           "platform": "KiaConnect",
           "email": "me@example.com",
           "password": "my-secret-password",
           "cars": [
             {
               "name": "Telluride",
               "vin": "XXXXXXXXXXXXXXXXX",
               "targetTemperature": "68",
               "refreshInterval": 3600000
             }
           ]
        }
    ]
}

```

## Setup Development Environment

To develop Homebridge plugins you must have Node.js 12 or later installed, and a modern code editor such as [VS Code](https://code.visualstudio.com/). This plugin template uses [TypeScript](https://www.typescriptlang.org/) to make development easier and comes with pre-configured settings for [VS Code](https://code.visualstudio.com/) and ESLint. If you are using VS Code install these extensions:

* [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

## Install Development Dependencies

Using a terminal, navigate to the project folder and run this command to install the development dependencies:

```
npm install
```

## Update package.json

Open the [`package.json`](./package.json) and change the following attributes:

* `name` - this should be prefixed with `homebridge-` or `@username/homebridge-` and contain no spaces or special characters apart from a dashes
* `displayName` - this is the "nice" name displayed in the Homebridge UI
* `repository.url` - Link to your GitHub repo
* `bugs.url` - Link to your GitHub repo issues page

When you are ready to publish the plugin you should set `private` to false, or remove the attribute entirely.

## Update Plugin Defaults

Open the [`src/settings.ts`](./src/settings.ts) file and change the default values:

* `PLATFORM_NAME` - Set this to be the name of your platform. This is the name of the platform that users will use to register the plugin in the Homebridge `config.json`.
* `PLUGIN_NAME` - Set this to be the same name you set in the [`package.json`](./package.json) file. 

Open the [`config.schema.json`](./config.schema.json) file and change the following attribute:

* `pluginAlias` - set this to match the `PLATFORM_NAME` you defined in the previous step.

## Build Plugin

TypeScript needs to be compiled into JavaScript before it can run. The following command will compile the contents of your [`src`](./src) directory and put the resulting code into the `dist` folder.

```
npm run build
```

## Link To Homebridge

Run this command so your global install of Homebridge can discover the plugin in your development environment:

```
npm link
```

You can now start Homebridge, use the `-D` flag so you can see debug log messages in your plugin:

```
homebridge -D
```

## Watch For Changes and Build Automatically

If you want to have your code compile automatically as you make changes, and restart Homebridge automatically between changes, you first need to add your plugin as a platform in `~/.homebridge/config.json`:
```
{
...
    "platforms": [
        {
            "name": "Config",
            "port": 8581,
            "platform": "config"
        },
        {
            "name": "<PLUGIN_NAME>",
            //... any other options, as listed in config.schema.json ...
            "platform": "<PLATFORM_NAME>"
        }
    ]
}
```

and then you can run:

```
npm run watch
```

This will launch an instance of Homebridge in debug mode which will restart every time you make a change to the source code. It will load the config stored in the default location under `~/.homebridge`. You may need to stop other running instances of Homebridge while using this command to prevent conflicts. You can adjust the Homebridge startup command in the [`nodemon.json`](./nodemon.json) file.

## Customise Plugin

You can now start customising the plugin template to suit your requirements.

* [`src/platform.ts`](./src/platform.ts) - this is where your device setup and discovery should go.
* [`src/platformAccessory.ts`](./src/platformAccessory.ts) - this is where your accessory control logic should go, you can rename or create multiple instances of this file for each accessory type you need to implement as part of your platform plugin. You can refer to the [developer documentation](https://developers.homebridge.io/) to see what characteristics you need to implement for each service type.
* [`config.schema.json`](./config.schema.json) - update the config schema to match the config you expect from the user. See the [Plugin Config Schema Documentation](https://developers.homebridge.io/#/config-schema).

## Versioning Your Plugin

Given a version number `MAJOR`.`MINOR`.`PATCH`, such as `1.4.3`, increment the:

1. **MAJOR** version when you make breaking changes to your plugin,
2. **MINOR** version when you add functionality in a backwards compatible manner, and
3. **PATCH** version when you make backwards compatible bug fixes.

You can use the `npm version` command to help you with this:

```bash
# major update / breaking changes
npm version major

# minor update / new features
npm version update

# patch / bugfixes
npm version patch
```

## Publish Package

When you are ready to publish your plugin to [npm](https://www.npmjs.com/), make sure you have removed the `private` attribute from the [`package.json`](./package.json) file then run:

```
npm publish
```

If you are publishing a scoped plugin, i.e. `@username/homebridge-xxx` you will need to add `--access=public` to command the first time you publish.

#### Publishing Beta Versions

You can publish *beta* versions of your plugin for other users to test before you release it to everyone.

```bash
# create a new pre-release version (eg. 2.1.0-beta.1)
npm version prepatch --preid beta

# publish to @beta
npm publish --tag=beta
```

Users can then install the  *beta* version by appending `@beta` to the install command, for example:

```
sudo npm install -g homebridge-example-plugin@beta
```


