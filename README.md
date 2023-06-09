# 4d-tips-deprecated-database-structure-options

The following database structure options are deprecated and will be edited or generate errors in the project conversion log file: 

|Deprecated feature	|Conversion status	|Comment|
|-|-|-|
|"Can't Modify" field option|	warning	|Automatically moved at form level during export to project|
|"Display only" field option|	warning	|Automatically moved at form level during export to project|
|"Mandatory" field option|	error	|Select "Reject NULL value input" option|

https://doc.4d.com/4Dv19R8/4D/19-R8/Converting-databases-to-projects.300-6125736.en.html#4361776

# case study

in this binary database we have a couple of mandatory fields.

<img width="440" alt="" src="https://github.com/miyako/4d-tips-deprecated-database-structure-options/assets/1725068/91347b41-617c-469a-86bf-140772524a45">

after conversion to project, a [log file](https://github.com/miyako/4d-tips-deprecated-database-structure-options/blob/main/example.4dbase/Logs/Conversion%202023-06-07T11-18-06.json) is generated.

despite the above quoted documentation, neither the warning nor error is included in the log file.

```json
{
	"messages": [],
	"success": true
}
```

# workaround

use the new [enhanced Xpath support](https://blog.4d.com/enhanced-xpath-support/) to update the catalog.

to check if standard Xpath is enabled or not: [is_xpath_enabled](example.4dbase/Project/Sources/Methods/is_xpath_enabled.4dm)

```4d
ASSERT(is_xpath_enabled)

$catalogFile:=File("/SOURCES/catalog.4DCatalog")
$dom:=DOM Parse XML source($catalogFile.platformPath)

If (OK=1)
	ARRAY TEXT($madatory_fields; 0)
	$madatory_field:=DOM Find XML element($dom; "/base/table/field/field_extra[@mandatory=true]"; $madatory_fields)
	For ($i; 1; Size of array($madatory_fields))
		$madatory_field:=$madatory_fields{$i}
		$field:=DOM Find XML element($madatory_field; "../")
		DOM SET XML ATTRIBUTE($field; "not_null"; True)
	End for 
	DOM EXPORT TO FILE($dom; $catalogFile.platformPath)
	DOM CLOSE XML($dom)
End if 
```

you need to restart the project for catalog changes to take place.

[RELOAD PROJECT](https://doc.4d.com/4Dv19/4D/19.6/RELOAD-PROJECT.301-6270050.en.html) does not work for the catalog.
