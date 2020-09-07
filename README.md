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

## Concho Functions

There are **three** Concho Functions:

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
          
        $Pieces[$i] = "\n".'<!-- Ashiva Console: Comparison Condition ( '.implode('`', $Equation_Parts).' ) cannot be parsed -->';
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
