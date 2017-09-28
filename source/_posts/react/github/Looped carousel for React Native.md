title: Looped carousel for React Native
date: 

categories: 
- react
- github
tags: 
- react
- github
---
# Looped carousel for React Native

[![NPM version](https://camo.githubusercontent.com/e169fb151edeefd52ed1dde01ed25e48b38d1038/687474703a2f2f696d672e736869656c64732e696f2f6e706d2f762f72656163742d6e61746976652d6c6f6f7065642d6361726f7573656c2e7376673f7374796c653d666c6174)](https://www.npmjs.com/package/react-native-looped-carousel) [![Build Status](https://camo.githubusercontent.com/ef78df4dd6cd61e9ce32d1c0eef147e9f58ec129/68747470733a2f2f7472617669732d63692e6f72672f617070696e7468656169722f72656163742d6e61746976652d6c6f6f7065642d6361726f7573656c2e737667)](https://travis-ci.org/appintheair/react-native-looped-carousel) [![Dependency Status](https://camo.githubusercontent.com/be3c252734a4f3993e65f4b551828ec3b17eeb70/68747470733a2f2f64617669642d646d2e6f72672f617070696e7468656169722f72656163742d6e61746976652d6c6f6f7065642d6361726f7573656c2e737667)](https://david-dm.org/appintheair/react-native-looped-carousel) [![devDependency Status](https://camo.githubusercontent.com/d7de83782716878c363e59dcac085085106e40bd/68747470733a2f2f64617669642d646d2e6f72672f617070696e7468656169722f72656163742d6e61746976652d6c6f6f7065642d6361726f7573656c2f6465762d7374617475732e737667)](https://david-dm.org/appintheair/react-native-looped-carousel#info=devDependencies)

Full-fledged "infinite" carousel for your next [react-native](https://github.com/facebook/react-native/) project. Supports iOS and Android.

Based on [react-native framework](https://github.com/facebook/react-native/) by Facebook.

## Demo

[![img](https://camo.githubusercontent.com/af681e3fe38da4aa45babe678ce8e10edc89edeb/687474703a2f2f7370726f6e696e2e6769746875622e696f2f696d672f72656163742e676966)](https://camo.githubusercontent.com/af681e3fe38da4aa45babe678ce8e10edc89edeb/687474703a2f2f7370726f6e696e2e6769746875622e696f2f696d672f72656163742e676966)

## Install

```
npm install react-native-looped-carousel --save
```

## Props

| Name                         | propType         | default value         | description                              |
| ---------------------------- | ---------------- | --------------------- | ---------------------------------------- |
| autoplay                     | boolean          | true                  | enables auto animations                  |
| delay                        | number           | 4000                  | number in milliseconds between auto animations |
| currentPage                  | number           | 0                     | allows you to set initial page           |
| pageStyle                    | style            | null                  | style for pages                          |
| contentContainerStyle        | style            | null                  | `contentContainerStyle` for the scrollView |
| onAnimateNextPage            | func             | null                  | callback that is called with 0-based Id of the current page |
| swipe                        | bool             | true                  | motion control for Swipe                 |
| **Pagination**               | ---              | ---                   | ---                                      |
| pageInfo                     | boolean          | false                 | shows `{currentPage} / {totalNumberOfPages}` pill at the bottom |
| pageInfoBackgroundColor      | string           | 'rgba(0, 0, 0, 0.25)' | background color for pageInfo            |
| pageInfoBottomContainerStyle | style            | null                  | style for the pageInfo container         |
| pageInfoTextStyle            | style            | null                  | style for text in pageInfo               |
| pageInfoTextSeparator        | string           | ' / '                 | separator for `{currentPage}` and `{totalNumberOfPages}` |
| **Bullets**                  | ---              | ---                   | ---                                      |
| bullets                      | bool             | false                 | wether to show "bullets" at the bottom of the carousel |
| bulletStyle                  | style            | null                  | style for each bullet                    |
| bulletsContainerStyle        | style            | null                  | style for the bullets container          |
| chosenBulletStyle            | stlye            | null                  | style for the selected bullet            |
| **Arrows**                   | ---              | ---                   | ---                                      |
| arrows                       | bool             | false                 | wether to show navigation arrows for the carousel |
| arrowsStyle                  | style            | null                  | style for navigation arrows              |
| arrowsContainerStyle         | style            | null                  | style for the navigation arrows container |
| leftArrowText                | string / element | 'Left'                | label / icon for left navigation arrow   |
| rightArrowText               | string / element | 'Right'               | label / icon for right navigation arrow  |

## Usage

```
import React, { Component } from 'react';
import {
  Text,
  View,
  Dimensions,
} from 'react-native';
import Carousel from 'react-native-looped-carousel';

const { width, height } = Dimensions.get('window');

export default class CarouselExample extends Component {

  constructor(props) {
    super(props);

    this.state = {
      size: { width, height },
    };
  }

  _onLayoutDidChange = (e) => {
    const layout = e.nativeEvent.layout;
    this.setState({ size: { width: layout.width, height: layout.height } });
  }

  render() {
    return (
      <View style={{ flex: 1 }} onLayout={this._onLayoutDidChange}>
        <Carousel
          delay={2000}
          style={this.state.size}
          autoplay
          pageInfo
          onAnimateNextPage={(p) => console.log(p)}
        >
          <View style={[{ backgroundColor: '#BADA55' }, this.state.size]}><Text>1</Text></View>
          <View style={[{ backgroundColor: 'red' }, this.state.size]}><Text>2</Text></View>
          <View style={[{ backgroundColor: 'blue' }, this.state.size]}><Text>3</Text></View>
        </Carousel>
      </View>
    );
  }
}
```

[Full example code](https://github.com/phil-r/react-native-looped-carousel/blob/master/Examples/Simple)

## Used in

- [React Native Buyscreen](https://github.com/appintheair/react-native-buyscreen)

## See also

- [React Native Grid Component](https://github.com/phil-r/react-native-grid-component)

------

More on react-native here: <http://facebook.github.io/react-native/docs/getting-started.html#content>