# inspectCapsule()
`inspectCapsule()` is a PHP function for inspecting:

 - an entire **Danis³h Capsule**; or
 - an individual **Danis³h Capsule Cell**
 
and will undertake the inspection regardless of whether the Danis3h Capsule in question is _active_ or _inactive_ on the current page.

______

## Example Queries

```php
// INSPECTS ENTIRE CAPSULE AT /ashiva/ashiva-menu/
inspectCapsule('<Ashiva_Menu (Ashiva) [#]>');

// INSPECTS ENTIRE CAPSULE AT /scotia-beauty/sb-company-details/
inspectCapsule('<SB_Company_Details (Scotia_Beauty) [#]>');

// INSPECTS STATIC CAPSULE CELL AT /ashiva/ashiva-menu/code/markup/button/ashiva-menu-button--html.json
inspectCapsule('<Ashiva_Menu (Ashiva) [@]Button___Ashiva_Menu_Button>');
inspectCapsule('<Ashiva_Menu (Ashiva) [@]Button___Ashiva_Menu_Button__HTML>');
inspectCapsule('<Ashiva_Menu (Ashiva) [@]Markup="Button___Ashiva_Menu_Button">');
inspectCapsule('<Ashiva_Menu (Ashiva) [@]Markup="Button___Ashiva_Menu_Button__HTML">');
inspectCapsule('<Ashiva_Menu (Ashiva) [@]Markup="button/ashiva-Menu-button">');
inspectCapsule('<Ashiva_Menu (Ashiva) [@]Markup="button/ashiva-Menu-button--html">');

// INSPECTS STATIC CAPSULE CELL AT /scotia-beauty/sb-company-details/code/styles/sb-company-details--css.json
inspectCapsule('<SB_Company_Details (Scotia_Beauty) [@]Styles>');
inspectCapsule('<SB_Company_Details (Scotia_Beauty) [@]SB_Company_Details__CSS>');
inspectCapsule('<SB_Company_Details (Scotia_Beauty) [@]Styles="SB_Company_Details">');
inspectCapsule('<SB_Company_Details (Scotia_Beauty) [@]Styles="SB_Company_Details__CSS">');

// INSPECTS DYNAMIC CAPSULE CELL AT /scotia-beauty/sb-company-details/code/markup/sb-company-details--php.php
inspectCapsule('<SB_Company_Details (Scotia_Beauty) [@]Markup="SB_Company_Details__PHP">');
```

______

## inspectCapsule Functions


### `getDocumentType()`

```php

function getDocumentType() {

  return (strpos(highlight_file(__FILE__, TRUE), '&lt;!DOCTYPE&nbsp;html&gt;') !== FALSE) ? 'HTML5' : 'Unknown';
}
```

### `inspectCapsule()` 

```php
function inspectCapsule($InspectionRequest) {

  $CellReference = $InspectionRequest;
  $InspectionRequest = str_replace('<', '&lt;', $InspectionRequest);
  $InspectionRequest = str_replace('>', '&gt;', $InspectionRequest);

  $CellReference = str_replace('<', '‹', $CellReference);
  $CellReference = str_replace('>', '›', $CellReference);
  $CellReference = trim(preg_replace('/\s+/', ' ', $CellReference));
  $CellReferenceArray = explode(' ', $CellReference);


  for ($i = 0; $i < count($CellReferenceArray); $i++) {
    
    switch (TRUE) {

      // CAPSULE NAME
      case (strpos($CellReferenceArray[$i], '‹') !== FALSE) :
      
        $CapsuleName = trim(str_replace(['‹', ' ', '›'], '', $CellReferenceArray[$i]));
        break;


      // CAPSULE PUBLISHER
      case (strpos($CellReferenceArray[$i], '(') !== FALSE) :
      
        $CapsulePublisher = trim(str_replace(['‹', '(', ' ', ')', '›'], '', $CellReferenceArray[$i]));
        break;
      
      
      // CAPSULE CELL
      case (strpos($CellReferenceArray[$i], '[@]') !== FALSE) :

        $FileTypes = ['Markup' => 'html', 'Styles' => 'css', 'Scripts' => 'js', 'Vectors' => 'svg', 'Data' => 'json'];
        $CellTypes = array_flip($FileTypes);

        $TargetCell = trim(str_replace(['‹', '"', ' ', '[@]', '›'], '', $CellReferenceArray[$i]));

        if (strpos($TargetCell, '=') === FALSE) {

          // ONLY CELLTYPE DECLARED
          if (in_array($TargetCell, ['Markup', 'Styles', 'Scripts', 'Vectors', 'Data'])) {
          
            $TargetCell .= '='.$CapsuleName;
          }
          
          // CELLTYPE NOT DECLARED. TARGETCELL FILETYPE DECLARED.
          elseif (in_array(url(array_reverse(explode('__', $TargetCell))[0]), array_keys($CellTypes))) {

            $TargetCell = $CellTypes[url(array_reverse(explode('__', $TargetCell))[0])].'='.$TargetCell;
          }

          // CELLTYPE NOT DECLARED. TARGETCELL FILETYPE NOT DECLARED.
          else {

            $TargetCell = (getDocumentType() === 'HTML5') ? 'Markup='.$TargetCell : 'Vectors='.$TargetCell;
          }
        }

        $TargetCellArray = explode('=', $TargetCell);

        $CellType = $TargetCellArray[0];
        $CellFilePathArray = explode('.', str_replace('--', '.', url($TargetCellArray[1])));
        
        $CellFilePath = explode('/', $CellFilePathArray[0]);
        $CellFileName = array_pop($CellFilePath);
        $CellFilePath = implode('/', $CellFilePath);

        $CellName = src($CellFileName);
        $CellName = (strtolower(substr($CellName, 0, strlen($CapsuleName))) === strtolower($CapsuleName)) ? substr($CellName, (strlen($CapsuleName) + 1)) : $CellName;
        $CellName .= ($CellName === $CellType) ? '' : '_'.$CellType;
        $CellName = (substr($CellName, 0, 1) === '_') ? substr($CellName, 1) : $CellName;

        $CellFileName = (in_array($CellName, array_keys($FileTypes))) ? $CapsuleName : $CellFileName;
        $CellFileType = (count($CellFilePathArray) > 1) ? $CellFilePathArray[1] : $FileTypes[$CellType];

        $CellModel = (in_array($CellFileType, ['php'])) ? 'Dynamic' : 'Static';

        break;
    }
  }

  // CHECK IF INSPECTION REQUEST IS FOR DEFAULT-MANIFESTED CAPSULE 
  if (strpos($CellReference, '[#]') !== FALSE) {
        
    $CellType = 'Capsule';
    $CellFilePath = 'Capsule';
    $CellFileName = 'Capsule';
    $CellFileType = 'Capsule';
  }

  // IDENTIFY INCOMPLETE INPUT
  switch (TRUE) {
   
    case (in_array(substr($CapsuleName, 0, 1), ['(', '['])) : echo '<h2>'.$InspectionRequest.'</h2><p>Please include Capsule Name</p>'; return; break;
    case (!isset($CapsulePublisher)) : echo '<h2>'.$InspectionRequest.'</h2><p>Please include Capsule Publisher</p>'; return; break;
    case (!isset($CellFileName)) : echo '<h2>'.$InspectionRequest.'</h2><p>Please include Capsule Cell</p>'; return; break;
  }

  // BUILD CAPSULE CELL URL
  $CapsuleCellURL = '';
  $CapsuleCellURL .= '/.assets/capsules';
  $CapsuleCellURL .= '/'.url($CapsulePublisher);
  $CapsuleCellURL .= '/'.url($CapsuleName);
  $CapsuleCellURL .= '/code/'.url($CellType);
  $CapsuleCellURL .= '/'.url($CellFilePath);
  $CapsuleCellURL .= '/'.url($CellFileName);
  $CapsuleCellURL .= '--'.url($CellFileType);
  $CapsuleCellURL .= (url($CellFileType) === 'php') ? '.php' : '.json';

  $CapsuleCellURL = str_replace('//', '/', $CapsuleCellURL);

  // GET CAPSULE STATUS ON PAGE
  $Page = page();
  $Page_Manifest_URL = str_replace('//', '/', $_SERVER['DOCUMENT_ROOT'].'/.assets/content/pages/'.url($Page).'/page.json');
  $PageManifest = json_decode(file_get_contents($Page_Manifest_URL), TRUE);

  $CapsuleCalled = FALSE;
  $CapsuleActive = 'Capsule Not Called by this Page';

  for ($i = 0; $i < count($PageManifest['Ashiva_Page_Build']['Modules']); $i++) {
    
    if (src($PageManifest['Ashiva_Page_Build']['Modules'][$i]['Module']) === src($CapsuleName)) {

      $CapsuleActive = $PageManifest['Ashiva_Page_Build']['Modules'][$i]['Active'] ?? TRUE;
      $CapsuleActive = ($CapsuleActive === TRUE) ? 'Capsule Active on this Page' : 'Capsule Currently Deactivated for this Page';
      break;
    }
  }

  // IF INSPECTION REQUEST IS FOR ENTIRE CAPSULE
  if ($CellType === 'Capsule') {
    
    $CapsuleSource = json_decode(requireCapsule($CapsuleName, $CapsulePublisher), TRUE);

    // DISPLAY CAPSULE
    $CapsuleInspection = [
      'Publisher' => $CapsulePublisher,
      'CapsuleName' => $CapsuleName,
      'CapsuleActive' => $CapsuleActive,
      'CapsulePath' => '/.assets/capsules/'.url($CapsulePublisher).'/'.url($CapsuleName).'/',
      'Source' => $CapsuleSource
    ];
  }

  // IF INSPECTION REQUEST IS FOR CAPSULE CELL
  else {

    $CapsuleCellSource = '⚠️ CapsuleCell does not exist';

    if (file_exists($_SERVER['DOCUMENT_ROOT'].$CapsuleCellURL)) {
    
      $CapsuleCellSource = json_decode(requireCapsule($CapsuleName, $CapsulePublisher), TRUE)[$CellType][$CellName];
    }

    // DISPLAY CAPSULE CELL
    $CapsuleInspection = [
      'Publisher' => $CapsulePublisher,
      'CapsuleName' => $CapsuleName,
      'CapsuleActive' => $CapsuleActive,
      'CellPath' => $CapsuleCellURL,
      'CellName' => $CellName,
      'CellType' => $CellType,
      'CellModel' => $CellModel,
      'CellFileName' => $CellFileName,
      'CellFileType' => $CellFileType,
      'Source' => $CapsuleCellSource
    ];
  }

  echo '<h2>'.$InspectionRequest.'</h2>'.print_r($CapsuleInspection, TRUE);
}

```
