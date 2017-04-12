# Custom Node.js LTS cartridge for OpenShift

![nodejs-openshift](https://cloud.githubusercontent.com/assets/581999/13374525/d25b0a28-dd8f-11e5-86cd-a49b3e698aa5.png)

This is a custom Node.js LTS cartridge that **takes care of auto-updating the Node.js and NPM versions** on each build.

## Why

Because the standard OpenShift cartridge never gets updated to the latest Node.js LTS version.

## When to use

When you need a quick and unsophisticated solution to run your application on the latest Node.js LTS version.

## How to

### The lazy way

Just click here:

[![Create Node.js app on OpenShift](https://cloud.githubusercontent.com/assets/581999/24554921/f1f2d5dc-1637-11e7-9089-d14791458744.png)](https://openshift.redhat.com/app/console/application_type/custom?&cartridges[]=https://raw.githubusercontent.com/GolamRagib/openshift-cartridge-nodejs/master/metadata/manifest.yml&name=appname)

### Using [web console](https://openshift.redhat.com/app/console/applications)

Go to [Choose a type of application](https://openshift.redhat.com/app/console/application_types) in your OpenShift Online account, paste the URL below into "Code Anything" textbox at the bottom of the page, click "Next", then set your public URL and click "Create Application".

    https://raw.githubusercontent.com/GolamRagib/openshift-cartridge-nodejs/master/metadata/manifest.yml

### Using command line

Assuming you have `rhc` installed (see [here](https://developers.openshift.com/en/managing-client-tools.html)), run:

    rhc app create appname https://raw.githubusercontent.com/GolamRagib/openshift-cartridge-nodejs/master/metadata/manifest.yml

…where `appname` is the name of your application.

If you want to create the app with **your own source code** instead of the provided application template, run:

    rhc app create appname \
      https://raw.githubusercontent.com/GolamRagib/openshift-cartridge-nodejs/master/metadata/manifest.yml \
      --from-code=https://github.com/you/your-repo.git

…where `https://github.com/you/your-repo.git` is your application repository URL.

Make sure your app has the appropriate `start` entry in `package.json` (see note below).

## Customizing the Node.js and npm versions

### Node.js

A different URL can be specified either via `NODE_VERSION_URL` environment variable or by setting `.openshift/NODE_VERSION_URL` marker in your application repository. For instance, you'd get the **latest 0.10.x** (0.10.41 as of Jan 24, 2015) by putting this in `NODE_VERSION_URL` variable or `.openshift/NODE_VERSION_URL` marker:

    https://semver.io/node/resolve/0.10

If you're using a non-default Node.js version and you're planning to **scale the application across multiple gears**, you **must use the environment variable** (learn [here](https://github.com/icflorescu/openshift-cartridge-nodejs/issues/23) why).

### npm

A different npm version can be specified either via `NPM_VERSION_URL` environment variable or by setting `.openshift/NPM_VERSION_URL` marker in your application repository. For instance, you'd get the latest 2.x (2.14.16 as of Jan 24, 2016) by putting this in `NPM_VERSION_URL` variable or `.openshift/NPM_VERSION_URL` marker:

    https://semver.io/npm/resolve/2

## Features

- The cartridge [build script](https://github.com/GolamRagib/openshift-cartridge-nodejs/blob/master/bin/control#L24) is using [this function](https://github.com/GolamRagib/openshift-cartridge-nodejs/blob/master/lib/util#L3) to check for the latest Node.js and npm versions and installs them if necessary;
- This cartridge **can** be scaled;
- This cartridge **does not** (yet?) have a hot-deploy strategy.

## Notes

- Can't guarantee this cartridge is production-ready. Some people use it though (on **their own responsibility**).
- This is a lean cartridge. No unnecessary modules are installed. Which means that  unlike the standard Node.js cartridge – it won't install [supervisor](https://github.com/isaacs/node-supervisor) for you. You'll have to implement your own logic to auto-restart on errors. The [provided application template](https://github.com/icflorescu/openshift-cartridge-nodejs/blob/master/usr/template/start.js) is using [cluster](http://nodejs.org/api/cluster.html) for that.
- The cartridge **emulates** the execution of `npm start` to start your application, so **make sure your application entrypoint is defined in your start script of your `package.json` file**. See [`package.json` in the provided template](https://github.com/GolamRagib/openshift-cartridge-nodejs/blob/master/usr/template/package.json) or read the [`npm` docs](https://docs.npmjs.com/cli/start) for more information. See a discussion [here](https://github.com/icflorescu/openshift-cartridge-nodejs/issues/32) to learn why the cartridge is **emulating** `npm start` instead of actually running it.
- The cartridge also **emulates** the execution of `npm stop` **if** the script is found in your `package.json` file. Otherwise, a `SIGTERM` is sent to the running process using `kill`.
- Due to OpenShift's outdated gcc (**4.4.7** as of Jan 4 2016), **native modules (such as [pg-native](https://github.com/brianc/node-pg-native)) won't compile on recent Node.js versions. They'll only work on Node.js 0.10 and 0.12**. See [this issue](https://github.com/icflorescu/openshift-cartridge-nodejs/issues/12) for more info.
- Upon cartridge initialization, the Node.js binary package is downloaded and installed, which **may take a while**, depending on OpenShift server and network load. 2 - 3 minutes is quite often, but 5 - 10 minutes is not uncommon, especially for scalable multi-gear setups (if you specified "Scale with web traffic").
- The cartridge automatically installs the npm `dependencies` as needed on each build/deploy event; `devDependencies` are not installed.

## Credits

See contributors [here](https://github.com/icflorescu/openshift-cartridge-nodejs/graphs/contributors).

## License

The [ISC License](https://github.com/icflorescu/openshift-cartridge-nodejs/blob/master/LICENSE).
