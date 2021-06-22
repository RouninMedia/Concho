# Concho
**Concho** is an Ashiva shorthand for ***con**ditionally* *e**cho**-ing* snippets of HTML markup.

Concho statements are simply shorthand representations of ternary echo statements in raw PHP. The two statements below are the same:

 - **Ternary** `echo` **statement:** `echo (($Impressum['Credits']['Active'] === TRUE) && ($Impressum['Credits']['Web']['Active'] === TRUE)) ? '<meta property="author" content="'.$Impressum['Credits']['Web']['Author']['Name'].'" />'."\n" : '';`
 - `concho` **statement:** `echo concho('<meta name="author" content="|Credits::Web::Author::Name|" />'."\n", [$Impressum], ['Credits::Active', 'Credits::Web::Active']);`

A **Concho** statement conditionally echoes:

 - single HTML element attributes
 - single lines of HTML

or:

 - blocks of HTML

and *also*:

 - `danis3hModule HTML Components`
 - `danis3hModule SVG Components`

HTML markup snippets conditionally echoed by **Concho** usually contain data extracted from an Ashiva `PageManifest` or `SiteManifest`.

________

## Concho ParseStrings

When the `concho()` function is invoked, it may, optionally, reference one or more PHP Associative Array `Data Sources`.

Any `value` may be easily extracted from a `Data Source` using **Concho ParseString** syntax.

eg. `Impressum::Credits::Web::Author::Name`

A **Concho ParseString** is a string which, when parsed, describes the location of any `value` within a PHP Associative Array.

For instance, the **ParseString** above, entirely straightforwardly, describes the following:

`$Data_Source['Impressum']['Credits']['Web']['Author']['Name']`

**Concho ParseStrings** may appear in **three** different contexts in the `concho()` function:

 1. in the **HTML** to be conditionally echoed, surrounded by pipes, like this: `|Impressum::Credits::Web::Author::Name|`
 2. as an *entire **Concho Condition***, where the `value` the **ParseString** describes is a `boolean`, like this: `'Impressum::Credits::Web'`
 3. as *part* of a ***Concho Condition***, where the Concho Condition is a **Comparison**, like this: ``'Asset::As`===`font'``

________

## Concho ParseString Prefixes

All **ParseStrings** have a numerical prefix, either implicit or explicit.

So far, we have only seen **ParseStrings** without explicit numerical prefixes - these all have an implicit or default prefix of `0`.

What this means in practice is that unprefixed **ParseStrings** are identical to `0`-prefixed **ParseStrings**:

 - `0::Impressum::Credits::Web::Active`
 - `Impressum::Credits::Web::Active`

The two **ParseStrings** immediately above are identical.

Numerical prefixes higher than `0` **must** be explicitly stated:

 - `1::Impressum::Credits::Web::Active`
 - `2::Impressum::Credits::Web::Active`
 - `3::Impressum::Credits::Web::Active` etc.
 
The numerical prefix references the **Index** of the `Data Source`, the **ParseString** applies to.

________

## Working Example of Concho

```
$SiteManifest = [

  'Global' => [

    'Test' => [

      'Path' => 'Test Successful'
    ]
  ]
];

$PageManifest = [
    
    'Asset' => [
      
      'Preload' => TRUE,
      'Type' => 'font/woff2',
      'As' => 'font',
      'URL' => '/.assets/theme/elements/fonts/scotia-beauty.woff2',
    ],
    
    'Impressum' => [
        
        'Credits' => [
            
            'Web' => [
                
                'Active' => TRUE,
                
                'Author' => [
                    
                    'Name' => 'Scotia Beauty'
                ]
            ]
        ]
    ]
];

$Other_Source = [];

echo concho('<meta name="author" content="|Impressum::Credits::Web::Author::Name|" />'."\n", [$PageManifest], ['Impressum::Credits::Web::Active']);
echo concho('<meta name="author" content="|1::Impressum::Credits::Web::Author::Name|" />'."\n\n", [$Other_Source, $PageManifest], ['1::Impressum::Credits::Web::Active', (5 > 4)]);
echo concho('<link rel="preload" href="|Asset::URL|" as="|Asset::As|" type="|Asset::Type|" '.concho('crossorigin="anonymous" ',  ['Asset::As`===`font', 'Asset::As`===`fontt'], [$PageManifest], 'OR').'/>'."\n\n", [$PageManifest], []);
echo concho('<p data-type="|1::Asset::Type|" data-test="Attribute |0::Global::Test::Path|">Element |0::Global::Test::Path|</p>', [$SiteManifest, $PageManifest], []);
```
________

## Concho Functions

There are **three** functions which make up **Concho**:

 - `concho()`
 - `conchoParse()`
 - `conchoQuery()`
 
 See below:
 
 ### `concho($Markup, $Data_Sources, $Conditions = [], $Logic = 'AND')`
 ```
 function concho($Markup, $Data_Sources, $Conditions = [], $Logic = 'AND') {
  
  if (strpos($Markup, '::') !== FALSE) {

    $Markup = explode('|', $Markup);
    $Markup = conchoParse($Markup, $Data_Sources);
    $Markup = implode('', $Markup);
  }
  
  // PARSE CONDITIONS
  $Conditions = conchoParse($Conditions, $Data_Sources);
  
  // CYCLE THROUGH BOOLEAN CONDITIONS
  for ($i = 0; $i < count($Conditions); $i++) {

    // CONDITION IS AN ASHIVA CONSOLE MESSAGE
    if (!is_bool($Conditions[$i])) {
      
      $Markup = $Conditions[$i];
      $Conditions[$i] = TRUE;
    }
    
    if ($Logic === 'AND') {
    
      $Markup = ($Conditions[$i] === FALSE) ? '' : $Markup;
    }
    
    elseif ($Logic === 'OR') {

      if ($Conditions[$i] === TRUE) break;
          
      elseif (($i === (count($Conditions) - 1)) && ($Conditions[$i] === FALSE)) {
        
        $Markup = '';
      }
    }
  }

  return $Markup;
}
 ```
 
 ### `conchoParse($Pieces, $Data_Sources)`
 ```
 function conchoParse($Pieces, $Data_Sources) {

  for ($i = 0; $i < count($Pieces); $i++) {

    // PARSE COMPARISON EQUATION CONDITIONS
    if (strpos($Pieces[$i], '`') !== FALSE) {

      $Equation_Parts = explode('`', $Pieces[$i]);
      
      if (count($Equation_Parts) <> 3) {
        
        $Pieces[$i] = '';
        $Pieces[$i] .= "\n";
        $Pieces[$i] .= '<!-- Ashiva Console: Comparison Condition ';
        $Pieces[$i] .= '( '.implode('`', $Equation_Parts).' ) ';
        $Pieces[$i] .= 'cannot be parsed -->';
        
        continue;
      }
      
      for ($j = 0; $j < count($Equation_Parts); $j++) {
          
        if (strpos($Equation_Parts[$j], '::') !== FALSE) {

          $First_Step = explode('::', $Pieces[$i])[0];
          $Data_Source = (is_numeric($First_Step)) ? $Data_Sources[$First_Step] : $Data_Sources[0];
          $Equation_Parts[$j] = conchoQuery($Equation_Parts[$j], $Data_Source);
        }
      }
      
      switch ($Equation_Parts[1]) {
          
        case ('==') : $Pieces[$i] = ($Equation_Parts[0] == $Equation_Parts[2]); break;
        case ('===') : $Pieces[$i] = ($Equation_Parts[0] === $Equation_Parts[2]); break;
        case ('!=') : $Pieces[$i] = ($Equation_Parts[0] != $Equation_Parts[2]); break;
        case ('<>') : $Pieces[$i] = ($Equation_Parts[0] <> $Equation_Parts[2]); break;
        case ('!==') : $Pieces[$i] = ($Equation_Parts[0] !== $Equation_Parts[2]); break; 
        case ('>') :  $Pieces[$i] = ($Equation_Parts[0] > $Equation_Parts[2]); break;
        case ('<') :  $Pieces[$i] = ($Equation_Parts[0] < $Equation_Parts[2]); break;
        case ('>=') : $Pieces[$i] = ($Equation_Parts[0] >= $Equation_Parts[2]); break;
        case ('<=') : $Pieces[$i] = ($Equation_Parts[0] <= $Equation_Parts[2]); break;  
        case ('<=>') : $Pieces[$i] = ($Equation_Parts[0] <=> $Equation_Parts[2]); break;
      }
    }
 
    else if (strpos($Pieces[$i], '::') !== FALSE) {

      $First_Step = explode('::', $Pieces[$i])[0];
      $Data_Source = (is_numeric($First_Step)) ? $Data_Sources[$First_Step] : $Data_Sources[0];
      $Pieces[$i] = conchoQuery($Pieces[$i], $Data_Source);
    }
  }

  return $Pieces;
}
 ```
 
 ### `conchoQuery($Query, $Data_Source)`
 ```
 function conchoQuery($Query, $Data_Source) {

  $Query_Response = $Data_Source;
  $Query = explode('::', $Query);
  $Data_Source_Index = (is_numeric($Query[0])) ? array_shift($Query) : 0;

  for ($j = 0; $j < count($Query); $j++) {
  
    if (!isset($Query_Response[$Query[$j]])) {
        
      return '<!-- Ashiva Console :: Invalid Query Step ['.$Query[$j].'] in Source '.$Data_Source_Index.' -->';
    }
      
    $Query_Response = $Query_Response[$Query[$j]];     
  }
  
  return $Query_Response;
}
 ```
_______

- Think through: *I DO like the idea of including and running a single Scaffold_Head() function in the Main Scaffold*
- Set up prototype Scaffold with `concho()`
- TIME TEST new Scaffold with `concho()`
