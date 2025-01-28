## Debug Character Data Command

Trigger Type: `Regex`
Trigger: `\Ag!debug char`

Response:
```go
{{$args := parseArgs 1 "" 
	(carg "string" "Character")
	(carg "string" "Environment")
}}
{{$char := lower ($args.Get 0)}}
{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$validChars := cslice}}
{{$env := or ($args.Get 1) "prod"}}
{{$validEnvs := cslice "prod" "dev"}}
{{$chosen := ""}}

{{if eq $env "dev"}}
	{{$characterData = (dbGet 98116114 "newCharTestData").Value}}
	{{sendMessage nil "Using Dev Character DB"}}
{{else if eq $env "prod"}}
	{{sendMessage nil "Using Production Character DB"}}
{{else if not (in $validEnvs $env)}}
	{{sendMessage nil "Invalid environment! Valid: `prod`, `dev`\nDefaulting to production\n"}}
{{end}}

{{range $key, $_ := $characterData}}
  {{$validChars = $validChars.Append (lower $key)}}
{{end}}

{{if not (in $validChars $char)}}
	{{sendMessage nil "Invalid Character entered!\n-# Check `g!chars` for valid Characters"}}
{{else if (in $validChars $char)}}
	{{$x := index $characterData $char }}
	{{$embed := cembed
		"title" (print "Name: `" $x.name "`")
		"description" (print "Rarity: `" $x.rarity "`\nWeight/Chance: `" $x.weight "%`\nWorth: `" $x.worth "`\nDecimal Color Code: `" $x.color "`\n\n[Image URL](" $x.url ")")
		"image" (sdict "url" $x.url)
		"color" $x.color
	}}
	{{sendMessage nil $embed}}
{{else}}
	{{sendMessage nil "Error encountered, please try again"}}
{{end}}
```

## Debug List Characters Command

Trigger Type: `Regex`
Trigger: `\Ag!debug list`

Response:
```go
{{$args := parseArgs 0 "" (carg "string" "Environment")}}
{{$data := (dbGet 98116114 "characterData").Value}}
{{$env := or ($args.Get 0) "prod"}}
{{$validEnvs := cslice "prod" "dev"}}
{{$fields := cslice}}

{{if eq $env "dev"}}
	{{$data = (dbGet 98116114 "newCharTestData").Value}}
	{{sendMessage nil "Using Dev Character DB"}}
{{else if eq $env "prod"}}
	{{sendMessage nil "Using Production Character DB"}}
{{else if not (in $validEnvs $env)}}
	{{sendMessage nil "Invalid environment! Valid: `prod`, `dev`\nDefaulting to production\n"}}
{{end}}

{{$charlist := cslice}}
{{range $key, $char := $data}}
    {{$charlist = $charlist.Append (dict
      "name" $char.name
      "rarity" $char.rarity
      "weight" $char.weight
      "worth" $char.worth
      "url" $char.url
      "color" $char.color
    )}}
{{end}}

{{range $i, $el := $charlist}}
  {{range $j, $el2 := $charlist}}
    {{if and (gt (index $el "weight") (index $el2 "weight")) (lt $i $j)}}
      {{$tmp := index $charlist $i}}
      {{$charlist.Set $i (index $charlist $j)}}
      {{$charlist.Set $j $tmp}}
    {{end}}
  {{end}}
{{end}}

  {{range $charlist}}
    {{$fields = $fields.Append (sdict
      "name" " "
      "value" (print "-# " .rarity "\n**" .name "** (`" .weight "%`)\nWorth: `" .worth "`\nColor: `" .color "`\n[Image URL](" .url ")")
      "inline" true
    )}}
  {{end}}
  {{sendMessage nil (cembed
    "title" "Character list"
    "color" 8008351
    "fields" $fields
  )}}
```

## Debug Pull Command

Trigger Type: `Regex`
Trigger: `\Ag!debug pull`

Response:
```go
{{$args := parseArgs 0 "" (carg "string" "Environment")}}
{{$rewards := dbGet 98116114 "characterData"}}
{{$env := or ($args.Get 0) "prod"}}
{{$validEnvs := cslice "prod" "dev"}}

{{if eq $env "dev"}}
	{{$rewards = dbGet 98116114 "newCharTestData"}}
	{{sendMessage nil "Using Dev Character DB"}}
{{else if eq $env "prod"}}
	{{sendMessage nil "Using Production Character DB"}}
{{else if not (in $validEnvs $env)}}
	{{sendMessage nil "Invalid environment! Valid: `prod`, `dev`\nDefaulting to production\n"}}
{{end}}

{{$rand := div (toFloat (randInt 1000)) 10}}
{{$StrippedRep :=  slice (exec "rep") (add (len .User.Username) 4)}}
{{$rep := toFloat (reFind `\d+` $StrippedRep)}}
{{$buff := or (mult (div $rep 10) 2) (toFloat 0)}}
{{$calc := (toFloat (printf "%.1f" (sub $rand $buff)))}}
{{$chosen := ""}}

{{sendMessage nil (print "-# Buff = ( rep:`" $rep "` / 10) * 2) = `" $buff "`\n-# Calc = (rand:`" $rand "` - `" $buff "`) = `" $calc "`")}}

{{range $key, $value := $rewards.Value}}
  {{$calc = sub $calc (toFloat $value.weight)}}
  {{if le $calc (toFloat 0)}}
    {{$chosen = $value}}
    {{break}}
  {{end}}
{{end}}
{{sendMessage nil (print "-# Calc - Weight = `" (printf "%.1f" $calc) "`\nResult: **" $chosen.name "** (Chance: `" $chosen.weight "%`)")}}
```
