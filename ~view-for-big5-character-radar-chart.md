---
PARTIAL_VERSION: v1.5.1
---
# -

## Types

The following schema should be placed as a child of the metadata
```ts
type Tuple = type Array[number, number]
type CANOE<T> = {
    C: T;
    A: T;
    N: T;
    O: T;
    E: T;
}

type CharacterName = string
```

## Commits

// v1.5.1 refactor
// v1.5 Use dv variables in metadata
// v1.4 fixed bug with wrong max property name
// v1.3 fixed a bug with scores ordering

# =

# _

~~~dataviewjs
const VERSION = "v1.5.1";
const plugin = this.app.plugins.plugins["obsidian-echarts"]; //echart-instance
const vf = this.app.workspace.getActiveFile();
const MAX_SCORE = 100;
const WIDTH = 420,
      HEIGHT = 420;
//data
const {frontmatter} = this.app.metadataCache.getFileCache(vf);
const radarChartTitle = ` ${vf.basename}'s' Personality Chart ${VERSION}`;
const legendDatums = ["Canoe", "Canoe Value"];
const seriesName = "CANOE vs value";


// main entry program : use enhanced function
const renderChart = createRendererForChart.call(this, {
  frontmatter,
  seriesName,
  legendDatums,
  radarChartTitle,
});
this.app.workspace.onLayoutReady(renderChart.bind(this));

// enhance a function
function createRendererForChart(config) {
  // always remove the last child.
    this.container?.lastChild?.remove();
    const {
      frontmatter,
      seriesName,
      radarChartTitle,
      legendDatums,
    } = config;
    if (!frontmatter?.hasOwnProperty("CANOE")) {
    // create property warning for metadata not matching app arity.
        return () => dv.el(
            "p", 
            `${vf.basename} does not 
            have the required inputs.`
        ); // early out;
    }

  // populate data for renderer to use through closure.
  const { C, A, N, O, E } = frontmatter.CANOE;
  const radarDatums = [
    {
      scoreTuple: C,
      indicatorTuple: makeTraitTuple("Conscientious", "chaotic"),
    },
    {
      scoreTuple: A,
      indicatorTuple: makeTraitTuple("Agreeable", "asshole"),
    },
    {
      scoreTuple: N,
      indicatorTuple: makeTraitTuple("Nice", "neurotic"),
    },
    {
      scoreTuple: O,
      indicatorTuple: makeTraitTuple("Open-to-\nnew", "outistic"),
    },
    {
      scoreTuple: E,
      indicatorTuple: makeTraitTuple("Extrovert", "endothermic"),
    },
  ];
  return function renderer() {
    const [indicators, scores] = makeIndicatorsScoresTuple(
      radarDatums,
      // normalize data in scores.
      // weight the maxscore so 2x + 3y = 100
      (scoreTuple, option = manufactureIndicatorOption()) => {
        const [a, b] = scoreTuple;
        const { maxScore } = option;
        const multiplier = maxScore / (a + b);
        return scoreTuple.map((scoreTupleItem, idx) =>
          (scoreTupleItem * multiplier).toFixed(2)
        );
      }
    );

    // arrange the scores and indicators so they are alternately adjacent from each other.
    const _scores = sortAlternateAdjacent(scores);
    const _indicators = sortAlternateAdjacent(indicators);
    const radarIndicators = _indicators.map((indicator) => ({
      ...indicator,
      max: indicator.maxScore,
    }));
    const _datas = [{
        value: _scores,
        name: radarChartTitle + " Series",
    }]

    const option = populateRadarOptionConfig({
      legendDatums,
      radarChartTitle,
      radarIndicators,
      _datas,
      seriesName,
    });

    plugin.render(option, this.container);
  };
}

function populateRadarOptionConfig(
  config = {
    legendDatums: {},
    radarChartTitle: "radarChartTitle",
    radarIndicators: [],
    _datas: [],
    seriesName: "",
  },
  stableSeriesConfig = {
    areaStyle: {},
  },
  stableConfig = {
    width: WIDTH,
    height: HEIGHT,
    tooltip: {
      trigger: "item",
    },
  },
  chartType = "radar"
) {
  const {
    legendDatums,
    radarChartTitle,
    radarIndicators,
    _datas,
    seriesName,
  } = config;
  const option = {
    ...stableConfig,
    legend: {
      data: legendDatums,
    },
    title: {
      text: radarChartTitle,
    },

    radar: {
      indicator: radarIndicators,
    },
    series: {
      data: _datas,
      name: seriesName,
      type: chartType,
      ...stableSeriesConfig,
    },
    type: chartType,
  };
  return option;
}

/**
 * Sort indicators into two joined lists (even <-> odd )
 */
function sortAlternateAdjacent(indicators) {
  const odds = [];
  return indicators.map(sortToEvensAndOdds).filter(Boolean).concat(odds);

  function sortToEvensAndOdds(item, idx, items) {
    if (idx === 0) return item;

    if (idx % 2 !== 0) {
      // obtain odd into sideEffects;
      odds.push(item);
      return null;
    }
    return item;
  }
}

function makeTraitTuple(
  yangTrait,
  yinTrait,
  option = manufactureIndicatorOption()
) {
  return [
    manufactureIndicator(yangTrait, option),
    manufactureIndicator(yinTrait, option),
  ];
}
// ceate default config for makeTraitTuple
function manufactureIndicatorOption(option = {}) {
  return { maxScore: MAX_SCORE, ...option };
}
// create helper tuple for Trait.
function manufactureIndicator(
  name,
  option = manufactureIndicatorOption(option)
) {
  return {
    name,
    ...option,
  };
}

/**
 * update a context property inside an array of contexts
 */
function updateDatasByIndex(datas, idx, update) {
  if (idx >= datas.length || idx < 0) {
    throw new Error(
      JSON.stringify({ message: "idx is greater than length of array" })
    );
  }
  return datas.slice(0, idx).concat(
    [
      {
        ...datas.at(idx),
        ...update,
      },
    ],
    datas.slice(idx + 1)
  );
}

// create indicators and scores tuple
// tuple use the default ordering
// @return [flattened, flattend]
function makeIndicatorsScoresTuple(
  radarDatums,
  transformScoreTuple = (tuple) => tuple
) {
  const scores = [];
  const indicators = [];
  for (const [idx, radarDatum] of radarDatums.entries()) {
    const { scoreTuple, indicatorTuple } = radarDatum;
    indicators.push(indicatorTuple);
    const transformedScoreTuple = transformScoreTuple(
      scoreTuple,
      manufactureIndicatorOption()
    );
    scores.push(transformedScoreTuple);
  }
  return [indicators.flat(), scores.flat()];
}
~~~

# ---Transient5.0

```js
~~~dataviewjs
// uncomment the above
const VERSION = "v1.5.0"

const plugin = this.app.plugins.plugins['obsidian-echarts']
const vf = this.app.workspace.getActiveFile()
const MAX_SCORE = 100;

this.app.workspace.onLayoutReady(funco.bind(this))


//echart-instance

// data (iceberg)

function funco() {

const {frontmatter} = this.app.metadataCache
    .getCache(vf.path);

if (!frontmatter?.hasOwnProperty("CANOE")) {
    this.container?.lastChild?.remove();
    return dv.el("p", `${vf.basename} does not have the required inputs.` )
}
this.container?.lastChild?.remove();
const {C,A,N,O,E} = frontmatter?.CANOE
const canoe = [C,A,N,O,E];


const [_C,_A,_N,_O,_E] = canoe.map((stic) => {
    return stic
        .map(Number)
});

//data (quicksilver)
const legendDatums = ['Canoe', 'Canoe Value']
const seriesName = 'CANOE vs value';
const radarChartTitle = ` ${vf.basename}'s' Personality Chart ${VERSION}`;
const characters =  [
    {
      value: [],
      name: radarChartTitle + " Series"
    },
]

const radarDatums = [
    {
        scoreTuple: _C ,
        indicatorTuple: 
            makeTraitTuple("Conscientous", "chaotic"),
    },
    {
        scoreTuple: _A ,
        indicatorTuple: 
            makeTraitTuple("Agreeable", "asshole"),
    },
    {
        scoreTuple: _N ,
        indicatorTuple: 
            makeTraitTuple("Nice", "neurotic"),
    },
    {
        scoreTuple: _O ,
        indicatorTuple: 
            makeTraitTuple("Open-to-\nnew", "outistic"),
    },
    {
        scoreTuple: _E,
        indicatorTuple: 
            makeTraitTuple("Extrovert", "endothermic"),
    },
]



const [
    indicators, 
    scores
] = makeIndicatorsScoresTuple(
        radarDatums, 
        // normalize data in scores.
        // weight the maxscore so 2x + 3y = 100
        (
            scoreTuple,
            option = manufactureIndicatorOption()
        ) => {    
            const [a,b] = scoreTuple;
            const {maxScore} = option;
            const multiplier = maxScore / (a + b);
            return scoreTuple.map(
                (scoreTupleItem, idx) => (
                    scoreTupleItem * multiplier
                ).toFixed(2)
            )
        }
    )

// create indicators and scores tuple
// tuple use the default ordering
// @return [flattened, flattend]
function makeIndicatorsScoresTuple(
    radarDatums, 
    transformScoreTuple = (tuple) => tuple
) {
    const scores = [];
    const indicators = [];
    for (const [idx, radarDatum] of radarDatums.entries()) {
        const {scoreTuple, indicatorTuple} = radarDatum;
        indicators.push(indicatorTuple);
        const transformedScoreTuple = 
            transformScoreTuple(
                scoreTuple, manufactureIndicatorOption()
            )
        // console.log({transformedScoreTuple})
        scores.push(transformedScoreTuple)
    }
    return [indicators.flat(), scores.flat()]
}

// arrange the scores and indicators so they are alternately adjacent from each other.
const _scores = sortAlternateAdjacent(scores)
const _indicators = sortAlternateAdjacent(
    indicators,
)


const _datas = updateDatasByIndex(characters, 0, {
    value: _scores,
})
function updateDatasByIndex(datas, idx, update) {
    if(idx >= datas.length || idx < 0) {
        throw new Error(JSON.stringify({message: "idx is greater than length of array"}))
    }
    return datas
        .slice(0,idx)
        .concat(  
            [{
                ...datas.at(idx), 
                ...update
            }],
            datas.slice(idx + 1)
        )
}

// @debugcheck
//console.log({_datas,_indicators})
const option = {
    legend: {
        data: legendDatums
    },
    tooltip: {
        trigger: "item"
    },
    width: 600,
    height: 400,
    title: {
      text: radarChartTitle
    },
    radar: {
        indicator: _indicators.map((indicator) => {
            return {
                ...indicator,
                max: indicator.maxScore
            }
        }),
    },
    series: {
        areaStyle: {},
        name: seriesName,
        type: 'radar',
        data: _datas
    },
    type: 'radar'
};

const boundMain = main.bind(this)
setTimeout(boundMain)
function main() {
    plugin.render(option, this.container)
    console.log('main');
}

function sortAlternateAdjacent(indicators) {
    const sideEffects = []
    return indicators
        .map(sortToEvensAndOdds)
        .filter(Boolean).concat(sideEffects)
    
    function sortToEvensAndOdds(item, idx, items) {
        if (idx === 0) return item;
    
        if (idx % 2 !== 0) {  // odd
            sideEffects.push(item)
            return null
        } 
        return item;
    }
}

function makeTraitTuple(yangTrait, yinTrait,
    option = manufactureIndicatorOption()
) {
    return [
        manufactureIndicator(yangTrait, option),
        manufactureIndicator(yinTrait, option)
    ]
}

function manufactureIndicatorOption(option = {}) {
    return {maxScore: MAX_SCORE, ...option}
};
function manufactureIndicator(
    name, 
    option = manufactureIndicatorOption(option)
) {
    return {
        name, 
        ...option
    }
}
}
~~~
```

## v1.2

largely static unrefactored but working, creates a big 5 personality chart.
```js
~~~dataviewjs
//v1.2

//echart-instance
const plugin = app.plugins.plugins['obsidian-echarts']

//data
const MAX_SCORE = 100;
const HEIGHT = 400, WIDTH = 600;

const seriesName = 'CANOE vs value'
const characters =  [
    {
      value: [],
      name: 'McCannin Personality Chart v1.2'
    },
]
const radarDatums = [
    {
        scoreTuple: [5,2] ,
        indicatorTuple: 
            makeTraitTuple("Conscientous", "chaotic"),
    },
    {
        scoreTuple: [2,5] ,
        indicatorTuple: 
            makeTraitTuple("Agreeable", "asshole"),
    },
    {
        scoreTuple: [2,5] ,
        indicatorTuple: 
            makeTraitTuple("Nice", "neurotic"),
    },
    {
        scoreTuple: [5,2] ,
        indicatorTuple: 
            makeTraitTuple("Open-to-\nnew", "outistic"),
    },
    {
        scoreTuple: [5,2] ,
        indicatorTuple: 
            makeTraitTuple("Extrovert", "endothermic"),
    },
]

const [indicators, scores] = makeIndicatorsScoresTuple(
    radarDatums, (
        scoreTuple,
        option = manufactureIndicatorOption()
    ) => {    
        // weight the maxscore so that it is 2x + 3y = 100
        const [a,b] = scoreTuple;
        const {maxScore} = option;
        const multiplier = maxScore / (a + b);
        return scoreTuple.map(
            (scoreTupleItem) => (scoreTupleItem * multiplier).toFixed(2)
        )
    })


// entrydata for apache radar chart machine
// make final transformations here.
const _indicators = makeIndicators(indicators)
    .map((indicator) => {
        const savedMaxScore = indicator?.maxScore || 0
        if (indicator.maxScore) {
            delete indicator.maxScore;
        }
        indicator.max = savedMaxScore
        return indicator;
    })
console.log({_indicators})

function makeIndicators(indicators) {
    const sideEffects = []
    return indicators
        .map(sortToEvensAndOddsPred)
        .filter(Boolean)
        .concat(sideEffects)
        
    function sortToEvensAndOddsPred(item, idx, items) {
        if (idx === 0) return item;
    
        if (idx % 2 !== 0) {  // odd
            sideEffects.push(item)
            return null
        } 
        return item;
    }
}

function makeIndicatorsScoresTuple(radarDatums, transformScoreTuple = (tuple) => tuple) {
    //radarDatums must have scoreTuple, and indicatorTuple

    const scores = [];
    const indicators = [];
    for (const [idx, radarDatum] of radarDatums.entries()) {
        const {scoreTuple, indicatorTuple} = radarDatum;
        indicators.push(indicatorTuple);
        const transformedScoreTuple = 
            transformScoreTuple(
                scoreTuple, manufactureIndicatorOption()
            )
        scores.push(transformedScoreTuple)
    }
    return [indicators.flat(), scores.flat()]
}

function makeTraitTuple(
    yangTrait,
    yinTrait,
    option = manufactureIndicatorOption()
) {
    return [
        manufactureIndicator(yangTrait, option),
        manufactureIndicator(yinTrait, option)
    ]
}

const _datas = updateDatasByIndex(characters, 0, {
    value: scores,
})
console.log({_datas})
function updateDatasByIndex(datas, idx, update) {
    if(idx >= datas.length || idx < 0) {
        throw new Error(JSON.stringify({message: "idx is greater than length of array"}))
    }
    return datas
        .slice(0,idx)
        .concat(  
            [{
                ...datas.at(idx), 
                ...update
            }],
            datas.slice(idx + 1)
        )
}


const option = {
    tooltip: {},
    legend: {
        data: ['Canoe', 'Canoe Value']
    },
    width: WIDTH,
    height: HEIGHT,
 title: {
     text: 'prototype radar'
    },
    radar: {
        indicator: _indicators,
        triggerEvent: true,
    },
    series: {
        areaStyle: {},
        name: seriesName,
        type: 'radar',
        data: updateDatasByIndex(
            _datas,
            0, 
            {
            /*
                label: {
                    show: true,
                    formatter: function (params) {
                      return params.value;
                    }
                }
            */
            }
        )
    },
    type: 'radar'
};

const boundMain = main.bind(this)
setTimeout(boundMain)

const resizeConfig = {
    height: HEIGHT,
    width: "auto"
}
function main() {
    //plugin.render(option, this.container)

    if (!this?.$chart) {
        const echart = plugin.echarts();
        this.$chart = echart.init(this.container);
    }
    this.$chart.setOption(option);
    this.$chart.resize(resizeConfig);
}



function manufactureIndicatorOption(option = {}) {
    return {maxScore: MAX_SCORE, ...option}
};
function manufactureIndicator(
    name, 
    option = manufactureIndicatorOption(option)
) {
    return {
        name, 
        ...option
    }
}
~~~
