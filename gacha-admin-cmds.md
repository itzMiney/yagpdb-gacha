I would suggest to group all your admin commands in a Gacha Admin CC group

# Pulls

## Admin add Pulls Command
Trigger Type: `Regex`
Trigger: `\Aga!addpulls`

Response:
```go
{{$args := parseArgs 2 ""
  (carg "user" "User")
  (carg "int" "Pulls" 1)
}}
{{$target := userArg ($args.Get 0)}}
{{$pulls := $args.Get 1}}
{{sendMessage nil (print "Gave `" $pulls "` pulls to **" $target.Globalname "** (Now has `" (dbIncr $target.ID "pulls" $pulls) "`)")}}
```

## Admin remove Pulls Command
Trigger Type: `Regex`
Trigger: `\Aga!rempulls`

Response:
```go
{{$args := parseArgs 2 ""
  (carg "user" "User")
  (carg "int" "Pulls" 1)
}}
{{$target := userArg ($args.Get 0)}}
{{$pulls := $args.Get 1}}
{{$pullsDel := (print "-" ($args.Get 1))}}
{{sendMessage nil (print "Removed `" $pulls "` pulls from **" $target.Globalname "** (Now has `" (dbIncr $target.ID "pulls" $pullsDel) "`)")}}
```

## Admin set Pulls Command
Trigger Type: `Regex`
Trigger: `\Aga!setpulls`

Response:
```go
{{$args := parseArgs 2 ""
  (carg "user" "User")
  (carg "int" "Pulls" 1)
}}
{{$target := userArg ($args.Get 0)}}
{{$pulls := $args.Get 1}}
{{dbSet $target.ID "pulls" $pulls}}
{{sendMessage nil (print "Set Pulls for **" $target.Globalname "** to `" $pulls "`")}}
```

## Admin reset Pulls Command
Trigger Type: `Regex`
Trigger: `\Aga!restpulls`

Response:
```go
{{$args := parseArgs 1 ""
  (carg "user" "User")
}}
{{$target := userArg ($args.Get 0)}}
{{dbSet $target.ID "pulls" 0}}
{{sendMessage nil (print "Reset pulls for **" $target.Globalname "**")}}
```


## Admin reset Daily Cooldown Command
Trigger Type: `Regex`
Trigger: `\Aga!resetdaily`

Response:
```go
{{$args := parseArgs 1 ""
  (carg "user" "User")
}}
{{$target := userArg ($args.Get 0)}}
{{dbDel $target.ID "dailyClaimed"}}
{{sendMessage nil (print "Reset daily cooldown for **" $target.Globalname "**")}}
```

## Admin view Pulls Command
Trigger Type: `Regex`
Trigger: `\Aga!pulls`

Response:
```go
{{$args := parseArgs 2 "" (carg "user" "User")}}
{{$target := userArg ($args.Get 0)}}
{{$pulls := or (dbGet $target.ID "pulls").Value 0}}
{{sendMessage nil (print "**" $target.Globalname "** has `" $pulls "` pulls")}}
```

# Collection


## Admin Reset Collection Command
Trigger Type: `Regex`
Trigger: `\Aga!resetdaily`

Response:
```go
{{$args := parseArgs 1 "" (carg "user" "User")}}
{{$target := userArg ($args.Get 0)}}{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$characters := cslice}}
{{range $key, $_ := $characterData}}
  {{$characters = $characters.Append (lower $key)}}
{{end}}
{{range $characters}}
  {{dbDel $target.ID . }}
{{end}}
{{sendMessage nil (print "Reset Collection for **" $target.Globalname "**")}}
```

## Admin add Character to Collection Command
Trigger Type: `Regex`
Trigger: `\Aga!addchar`

Response:
```go
{{$args := parseArgs 3 ""
  (carg "string" "Character")
  (carg "int" "Amount" 1)
  (carg "user" "User") }}
{{$char := lower ($args.Get 0)}}
{{$amount := $args.Get 1}}
{{$target := userArg ($args.Get 2)}}
{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$validChars := cslice}}
{{range $key, $_ := $characterData}}    
  {{$validChars = $validChars.Append (lower $key)}}
{{end}}
{{if not (in $validChars)}}  
  {{sendMessage nil "Invalid Character entered!\n-# Check `g!chars` for valid Characters"}}
{{else}}
  {{$nonLowerChar := index $characterData $char}}	
  {{$charName := (index $nonLowerChar "name")}}
  {{sendMessage nil (print "Added `" $amount "` " $charName "'s to **" $target.Globalname "**'s Collection, now has `" (dbIncr $target.ID $char $amount) "` " $charName "'s")}}
{{end}}
```

## Admin remove Character from Collection Command
Trigger Type: `Regex`
Trigger: `\Aga!remchar`

Response:
```go
{{$args := parseArgs 3 ""  
  (carg "string" "Character")  
  (carg "int" "Amount" 1)  
  (carg "user" "User") }}
{{$char := lower ($args.Get 0)}}
{{$amount := $args.Get 1}}
{{$amountDel := (print "-" $amount)}}
{{$target := userArg ($args.Get 2)}}
{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$validChars := cslice}}
{{range $key, $_ := $characterData}}
  {{$validChars = $validChars.Append (lower $key)}}
{{end}}
{{if not (in $validChars)}}  
  {{sendMessage nil "Invalid Character entered!\n-# Check `g!chars` for valid Characters"}}
{{else}}
  {{$nonLowerChar := index $characterData $char}}
  {{$charName := (index $nonLowerChar "name")}}
  {{sendMessage nil (print "Removed`" $amount "` " $charName "'s from **" $target.Globalname "**'s Collection, now has `" (dbIncr $target.ID $char $amountDel) "` " $charName "'s left")}}
{{end}}
```

## Admin set Character amount for Collection Command
Trigger Type: `Regex`
Trigger: `\Aga!setchar`

Response:
```go
{{$args := parseArgs 3 ""
  (carg "string" "Character")
  (carg "int" "Amount" 1)
  (carg "user" "User")
 }}

{{$char := lower ($args.Get 0)}}
{{$amount := $args.Get 1}}
{{$target := userArg ($args.Get 2)}}{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$validChars := cslice}}
{{range $key, $_ := $characterData}}  
  {{$validChars = $validChars.Append (lower $key)}}
{{end}}

{{if not (in $validChars)}}
  {{sendMessage nil "Invalid Character entered!\n-# Check `g!chars` for valid Characters"}}
{{else}}
  {{$nonLowerChar := index $characterData $char}}
  {{$charName := (index $nonLowerChar "name")}}
  {{dbSet $target.ID $char $amount}}
  {{sendMessage nil (print "Set " $charName " for **" $target.Globalname "** to `" $amount "`")}}
{{end}}
```

## Admin reset Character for Collection Command
Trigger Type: `Regex`
Trigger: `\Aga!resetchar`

Response:
```go
{{$args := parseArgs 3 ""
  (carg "string" "Character")
  (carg "user" "User")
 }}

{{$char := lower ($args.Get 0)}}
{{$target := $args.Get 1}}
{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$validChars := cslice}}
{{range $key, $_ := $characterData}}
  {{$validChars = $validChars.Append (lower $key)}}
{{end}}
{{if not (in $validChars)}}
  {{sendMessage nil "Invalid Character entered!\n-# Check `g!chars` for valid Characters"}}
{{else}}
  {{$nonLowerChar := index $characterData $char}}
  {{$charName := (index $nonLowerChar "name")}}
  {{dbDel $target.ID $char}}	
  {sendMessage nil (print "Removed all " $charName "'s from **" $target.Globalname "**'s Collection")}}
{{end}}
```

## Admin reset Trade Command
Trigger Type: `Regex`
Trigger: `\Aga!resettrade`

Response:
```go
{{$args := parseArgs 1 ""
  (carg "user" "User")
}}
{{$target := userArg ($args.Get 0)}}
{{dbDel $target.ID "tradeOffer"}}
{{dbDel $target.ID "tradeChar"}}
{{dbDel $target.ID "tradeAmount"}}
{{dbDel $target.ID "tradePrice"}}
{{dbDel $target.ID "tradeInitiator"}}
{{sendMessage nil (print "Reset trade offers for **" $target.Globalname "**")}}
```

# Add/Update Characters Command

This command will create or update the database record for your gacha minigame with the characters inside the $characterData sdict
Use the command argument "dev" to generate/update the testing and development database entry, or "prod" to generate/update the database entry for live production
The default preset in this example are 14 Characters from the Anime/Manga Series "Bocchi the Rock!"

## Command
Trigger Type: `Regex`
Trigger: `\Aga!updatechars`

Response:
```go
{{$args := parseArgs 0 "" (carg "string" "Environment")}}
{{$env := or ($args.Get 0) "dev"}}
{{$validEnvs := cslice "prod" "dev"}}

{{$characterData := sdict
  "bocchi" (sdict "name" "Bocchi" "rarity" "Common" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/9/98/Hitori_Gotoh_Character_Design_2.png" 
    "color" 16746692 "weight" 40.0 "worth" 0.1)

  "ryo" (sdict "name" "Ryo" "rarity" "Uncommon" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/4/4a/Ryo_Yamada_Character_Design_2.png" 
    "color" 5921500 "weight" 15.0 "worth" 0.2)

  "kita" (sdict "name" "Kita" "rarity" "Rare" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/a/a8/Ikuyo_Kita_Character_Design_2.png" 
    "color" 14029630 "weight" 10.0 "worth" 0.4)

  "shima" (sdict "name" "Shima" "rarity" "Rare"
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/b/b6/Shima_Iwashita.png"
    "color" 3636653 "weight" 7.5 "worth" 0.45)

  "nijika" (sdict "name" "Nijika" "rarity" "Epic" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/9/92/Nijika_Ijichi_Character_Design_2.png" 
    "color" 16777023 "weight" 5.0 "worth" 0.5)

  "jimihen" (sdict "name" "Jimihen" "rarity" "Epic"
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/3/31/Jimihen.png"
    "color" 13609290 "weight" 4.5 "worth" 1)

  "seika" (sdict "name" "Seika" "rarity" "Mythic" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/0/0e/Seika_Ijichi.png" 
    "color" 14078853 "weight" 2.5 "worth" 1)

  "yoyoko" (sdict "name" "Yoyoko" "rarity" "Mythic"
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/8/8d/EP10_Yoyoko_Ohtsuki.png"
    "color" 5192485 "weight" 2.35 "worth" 1)

  "eliza" (sdict "name" "Eliza" "rarity" "Mythic"
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/0/07/Eliza_Shimizu.png" 
    "color" 16698767 "weight" 2.15 "worth" 3)

  "pa-san" (sdict "name" "PA-San" "rarity" "Super Mythic" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/a/a2/PA-san.png"
    "color" 1908021 "weight" 2.0 "worth" 4)

  "yuyu" (sdict "name" "Yuyu" "rarity" "Super Mythic" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/3/35/Yuyucolor.png"
    "color" 1837652 "weight" 1.7 "worth" 3) 

  "akubi" (sdict "name" "Akubi" "rarity" "Super Mythic"
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/d/d9/Akubicolor.png"
    "color" 10581468 "weight" 1.5 "worth" 5)

  "fuko" (sdict "name" "Fuko" "rarity" "Super Mythic"
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/5/57/Fuko_Honjo.png"
    "color" 12095842 "weight" 1.3 "worth" 5)

  "kikuri" (sdict "name" "Kikuri" "rarity" "Ultra Rare" 
    "url" "https://static.wikia.nocookie.net/bocchi-the-rock/images/8/88/Kikuri_Hiroi.png" 
    "color" 8008351 "weight" 0.5 "worth" 70)
}}

{{if eq $env "prod"}}
	{{dbSet 98116114 "characterData" $characterData}}
	{{sendMessage nil "Production Character data has been updated!"}}
{{else if eq $env "dev"}}
	{{dbSet 98116114 "newCharTestData" $characterData}}
	{{sendMessage nil "Dev Character data has been updated!"}}
{{else if not (in $validEnvs $env)}}
	{{sendMessage nil "Invalid environment! Valid: `prod`, `dev`"}}
{{end}}
```
