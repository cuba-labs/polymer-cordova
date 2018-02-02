This project is a demonstration on how to wrap CUBA Platform's Polymer Client into the hybrid mobile application
 using [Apache Cordova](https://cordova.apache.org/). 

The video recording of the corresponding webinar is available [here](https://www.cuba-platform.com/webinars/creating-hybrid-mobile-application). 

## Requirements
[Android SDK](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html#installing-the-requirements) and [Node.js](https://nodejs.org/en/download/) should be installed on your machine.

## Building Android Application
```bash
./gradlew buildCordova
```

## Step-by-step Guide
- [Create](https://doc.cuba-platform.com/manual-latest/polymer_in_studio.html) Polymer client using CUBA Studio
- Install [cordova cli](https://cordova.apache.org/docs/en/latest/guide/cli/index.html#installing-the-cordova-cli)
```bash
npm install -g cordova
```
- Create Cordova App using `cordova create` command:
```bash
cd <project dir>/modules/polymer-client/
cordova create cordova com.example.demo Demo
``` 

Cordova cli will generate required directory structure in the `cordova` directory. Skeletal web application will be 
generated in the `www` sub-directory. In our approach we will replace it by assembled Polymer client in a build time.

- Clear `www` directory 
```bash
rm -rf cordova/www/*
```

- Add `www` directory to `.gitignore`:
```bash
echo www/* > cordova/.gitignore
```

- Add Android Platform to our Cordova project:
```bash
cd cordova
cordova platform add android
```

- Add new build config to `polymer.json`:
```
  "builds": [
    ...
    {
      "name": "cordova",
      "preset": "es5-bundled",
      "addServiceWorker": false
    }
    ...
  ]
```

- Remove `<base>` tag from `index.html`. For other build targets it will be added by polymer tooling automatically 
according to the `basePath` config option.

- Add Cordova cli as a devDependency in `polymer-client/package.json`:
```
...
  "devDependencies": {
    ...
    "cordova": "^7.1.0"
  }
...
```

- Now it's time to tweak `polymerClientModule` in our main `build.gradle`:

The following task installs cordova tooling:
```groovy
task installCordovaPackages(type: NpmTask) {
    args = ['install']
    execOverrides {
        it.workingDir = 'cordova'
    }
}
```

The following tasks copies assembled Polymer client into the `cordova/www` folder:
```groovy
task prepareCordova(type: Copy, dependsOn: [assemble]) {
    from file('build/cordova')
    into "cordova/www"
}
```

The following task runs cordova cli in order to build mobile app:
```groovy
task buildCordova(type: NodeTask, dependsOn: [prepareCordova, installCordovaPackages]) {
    script = file('node_modules/cordova/bin/cordova')
    args = ['build']
    workingDir 'cordova'
}
```

Now you should be able to build mobile app using gradle:
```bash
./gradlew buildCordova
```
