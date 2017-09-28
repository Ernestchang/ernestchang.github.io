title: Looped carousel for React Native
date: 

categories: 
- react
- github
tags: 
- react
- github
---

# React Native Loading Spinner Overlay

[![NPM version](https://camo.githubusercontent.com/dd009b4c54b176e193c43f90b4403d52b939df4d/687474703a2f2f696d672e736869656c64732e696f2f6e706d2f762f72656163742d6e61746976652d6c6f6164696e672d7370696e6e65722d6f7665726c61792e7376673f7374796c653d666c6174)](https://npmjs.org/package/react-native-loading-spinner-overlay) [![NPM downloads](https://camo.githubusercontent.com/d8dad4a0bc467c538d7678baba8d5780775cdec1/687474703a2f2f696d672e736869656c64732e696f2f6e706d2f646d2f72656163742d6e61746976652d6c6f6164696e672d7370696e6e65722d6f7665726c61792e7376673f7374796c653d666c6174)](https://npmjs.org/package/react-native-loading-spinner-overlay) [![MIT License](https://camo.githubusercontent.com/4ad0bc4de8816451a4a76b886b76142b99d10ffb/687474703a2f2f696d672e736869656c64732e696f2f62616467652f6c6963656e73652d4d49542d626c75652e7376673f7374796c653d666c6174)](https://github.com/joinspontaneous/react-native-loading-spinner-overlay/blob/master/LICENSE)

> **tldr;** The only pure [React Native](https://facebook.github.io/react-native) Native iOS and Android loading spinner (progress bar indicator) overlay

[![Demo](https://camo.githubusercontent.com/8d5f6085477bef4cfee5ea16effcb9dd92b84b47/68747470733a2f2f63646e2e7261776769742e636f6d2f6e696674796c6574747563652f72656163742d6e61746976652d6c6f6164696e672d7370696e6e65722d6f7665726c61792f6d61737465722f6d656469612f64656d6f2e676966)](https://camo.githubusercontent.com/8d5f6085477bef4cfee5ea16effcb9dd92b84b47/68747470733a2f2f63646e2e7261776769742e636f6d2f6e696674796c6574747563652f72656163742d6e61746976652d6c6f6164696e672d7370696e6e65722d6f7665726c61792f6d61737465722f6d656469612f64656d6f2e676966)

## Index

- [Install](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#install)
- [Usage](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#usage)
- [Platforms](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#platforms)
- [Notes](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#notes)
- [Development](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#development)
- [Contributors](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#contributors)
- [Credits](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#credits)
- [License](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#license)

## Install

For React Native version `>=0.28.x` use version `>=0.3.x` (`0.2.x` is broken, sorry!):

```
npm install --save react-native-loading-spinner-overlay@latest
```

For React Native version `<=0.27.x` use version `0.1.x`:

```
npm install --save react-native-loading-spinner-overlay@0.1.x
```

## Usage

This usage shows the default styles and properties.

| Property     | Type                       | Default               | Description                              |
| ------------ | -------------------------- | --------------------- | ---------------------------------------- |
| cancelable   | `boolean`                  | `false`               | **Android**: If set to false, it will prevent spinner from hiding when pressing the hardware back button. If set to true, it will allow spinner to hide if the hardware back button is pressed. |
| color        | `string`                   | `white`               | Changes the spinner's color (example values are `red`, `#ff0000`, etc). For adjusting the contrast see `overlayColor` prop below. |
| animation    | `none`, `slide`, `fade`    | `none`                | Changes animation on show and hide spinner's view. |
| overlayColor | `string`                   | `rgba(0, 0, 0, 0.25)` | Changes the color of the overlay.        |
| size         | `small`, `normal`, `large` | `large`               | Sets the spinner's size. No other cross-platform sizes are supported right now. |
| textContent  | `string`                   | `""`                  | Optional text field to be shown.         |
| textStyle    | `style`                    | `-`                   | The style to be applied to the `<Text>` that displays the `textContent`. |
| visible      | `boolean`                  | `false`               | Controls the visibility of the spinner.  |

You can also add a child view to act as a custom activity indicator.

```
import React, { View, Text } from 'react-native';

import Spinner from 'react-native-loading-spinner-overlay';

class MyComponent extends React.Component {

  constructor(props) {
    super();
    this.state = {
      visible: false
    };
  }

  /* eslint react/no-did-mount-set-state: 0 */
  componentDidMount() {
    setInterval(() => {
      this.setState({
        visible: !this.state.visible
      });
    }, 3000);
  }

  render() {
    return (
      <View style={{ flex: 1 }}>
        <Spinner visible={this.state.visible} textContent={"Loading..."} textStyle={{color: '#FFF'}} />
      </View>
    );
  }
}
```

To use a custom activity indicator just pass it as child of the component:

```
<Spinner visible={this.state.visible}>
  <Text>This is my custom spinner</Text>
</Spinner>
```

## Platforms

> For `>= 0.3.x`:

- We use `ActivityIndicator` now!

> For `0.2.x`:

- Do not use this version due to [#22](https://github.com/niftylettuce/react-native-loading-spinner-overlay/issues/22), use `>= 0.3.x` please!

> For `<= 0.1.x`:

- iOS: this platform uses `Modal` ([docs](https://facebook.github.io/react-native/docs/modal.html)/[source](https://github.com/facebook/react-native/blob/master/Libraries/Modal/Modal.js)) to overlay and `ActivityIndicatorIOS` ([docs](https://facebook.github.io/react-native/docs/activityindicatorios.html)) for the loading spinner
- Android: this platform uses `Portal` ([source](https://github.com/facebook/react-native/blob/master/Libraries/Portal/Portal.js)) to overlay and `ActivityIndicator` ([docs](https://facebook.github.io/react-native/docs/activityindicator.html)) for the loading spinner

## Notes

> For `>= 0.3.x`:

- We use `ActivityIndicator` now!

> For `0.2.x`:

- This version is broken due to a dependency issue, see issue [#22](https://github.com/niftylettuce/react-native-loading-spinner-overlay/issues/22)

> For `<= 0.1.x`:

- Docs don't exist yet for `Portal`, see [this issue on GitHub](https://github.com/facebook/react-native/issues/2501); once those are in, then we can add a link to the source in [Platforms](https://github.com/joinspontaneous/react-native-loading-spinner-overlay#platforms)
- Until a release of React Native is shipped [for this pull request](https://github.com/facebook/react-native/pull/4974), Android's `ProgressBarAndroid`will not have support for a `StyleAttr` value of `"Normal"` - therefore we only support a `size`prop of `"small"` or `"large"` right now (defaulting to `"large"`) - in other words, we can only support Android's inverse styling with a `styleAttr` of `"Inverse"`, `"SmallInverse"` (for a `size` prop of `"small"`), and `"LargeInverse"` (for a `size` prop of `"large`") (since there is no `"Normal"` support right now for `"size"` of `"normal"`).

## Development

1. Fork/clone this repository
2. Run `npm install`
3. Make changes in `src` directory
4. Run `npm test` when you're done
5. Submit a pull request

## Contributors

- Nick Baugh [niftylettuce@gmail.com](mailto:niftylettuce@gmail.com)

## License

[MIT](https://github.com/joinspontaneous/react-native-loading-spinner-overlay/blob/master/LICENSE)