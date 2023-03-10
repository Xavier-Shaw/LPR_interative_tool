# Lean Privacy Review

## How to use it
Download the code and there are two ways to use the tool.

#### Vue Version (Add data and Show data)
1. Indicate your concerns with scores and reasons in the form
2. Click `Submit the Form` and replace the `data.json` in directory `/public` with the newest data.
3. Click `Generate Grid Chart` to see the overview of results (You might have to refresh the page after replacing the data).
4. Click `Reset` to reinitialize the grid chart
5. Click `Show Next Group` to show the privacy concerns of next group of users

#### Pure HTML+JS Version (Show data)
1. Click `Load Data` to load data from `data.json`, which is in the same directory as `gridChart.html`
2. Click `Generate Grid Chart` to see the overview of results
3. Click `Reset` to reinitialize the grid chart
4. Click `Show Next Group` to show the privacy concerns of next group of users

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
