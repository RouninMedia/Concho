# Concho
**Concho** is an Ashiva shorthand for ***con**ditionally* *e**cho**-ing* snippets of HTML markup.

In this case *snippets of HTML markup* usually refers to:

 - lines of HTML
 - single HTML element attributes

but can also mean:

 - `ashivaModule HTML Components`
 - `ashivaModule SVG Components`

HTML markup snippets conditionally echoed by **Concho** often contain data extracted from an Ashiva `PageManifest` or `SiteManifest`.

________

## Concho ParseStrings

When the `concho()` function is invoked, it may, optionally, reference one or more PHP Associative Array `Data Sources`.

Any `value` may be easily extracted from a `Data Source` using **Concho ParseString** syntax.

eg. `Impressum::Credits::Web::Author::Name`

A **Concho ParseString** is a string which, when parsed, describes the location of any `value` within a PHP Associative Array.

For instance, the **ParseString** above, entirely straightforwardly, describes the following:

`$Data_Source['Impressum']['Credits']['Web']['Author']['Name']`

**Concho ParseStrings** may appear in three different contexts in the `concho()` function:

 - in the **HTML** to be conditionally echoed, surrounded by pipes, like this: `|Impressum::Credits::Web::Author::Name|`
 - as an entire Concho Condition, where the `value` the **ParseString** describes is a `boolean`, like this: `'Impressum::Credits::Web'`
 - as part of a Concho Condition, where the Concho Condition is a **Comparison Equation**, like this: ``'Asset::As`===`font'``

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

echo concho('<meta name="author" content="|Impressum::Credits::Web::Author::Name|" />'."\n", ['Impressum::Credits::Web::Active'], [$PageManifest]);
echo concho('<meta name="author" content="|1::Impressum::Credits::Web::Author::Name|" />'."\n\n", ['1::Impressum::Credits::Web::Active', (5 > 4)], [[], $PageManifest]);
echo concho('<link rel="preload" href="|Asset::URL|" as="|Asset::As|" type="|Asset::Type|" '.concho('crossorigin="anonymous" ',  ['Asset::As`===`font', 'Asset::As`===`fontt'], [$PageManifest], 'OR').'/>'."\n\n", [], [$PageManifest]);
echo concho('<p data-type="|1::Asset::Type|" data-test="Attribute |0::Global::Test::Path|">Element |0::Global::Test::Path|</p>', [], [$SiteManifest, $PageManifest]);
```
________

## Concho Functions

There are **three** functions which make up **Concho**:

 - `concho()`
 - `conchoParse()`
 - `conchoQuery()`
 
 See below:
 
 ### `concho($Markup, $Conditions = [], $Sources = [[]], $Logic = 'AND')`
 ```
 function concho($Markup, $Conditions = [], $Sources = [[]], $Logic = 'AND') {
  
  if (strpos($Markup, '::') !== FALSE) {

    $Markup = explode('|', $Markup);
    $Markup = conchoParse($Markup, $Sources);
    $Markup = implode('', $Markup);
  }
  
  // PARSE CONDITIONS
  $Conditions = conchoParse($Conditions, $Sources);
  
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
 
 ### `conchoParse($Pieces, $Sources)`
 ```
 function conchoParse($Pieces, $Sources) {

  for ($i = 0; $i < count($Pieces); $i++) {

    // PARSE COMPARISON OPERATOR EQUATIONS
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
          $Source = (is_numeric($First_Step)) ? $Sources[$First_Step] : $Sources[0];
          $Equation_Parts[$j] = conchoQuery($Equation_Parts[$j], $Source);
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
      $Source = (is_numeric($First_Step)) ? $Sources[$First_Step] : $Sources[0];
      $Pieces[$i] = conchoQuery($Pieces[$i], $Source);
    }
  }

  return $Pieces;
}
 ```
 
 ### `conchoQuery($Query, $Source)`
 ```
 function conchoQuery($Query, $Source) {

  $Query_Response = $Source;
  $Query = explode('::', $Query);
  $Source_Index = (is_numeric($Query[0])) ? array_shift($Query) : 0;

  for ($j = 0; $j < count($Query); $j++) {
  
    if (!isset($Query_Response[$Query[$j]])) {
        
      return '<!-- Ashiva Console :: Invalid Query Step ['.$Query[$j].'] in Source '.$Source_Index.' -->';
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
