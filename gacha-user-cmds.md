I would suggest to group all of these commands in a CC Group for User Gacha Commands

# Core Commands

## Reset Daily Cooldown Crontab
Trigger Type: `Crontab`
Crontab: `0 0 * * *`

Channel: `#gacha`

Response:
```go
{{sendMessage nil (print "**New Dailies available!**\nTotal Dailies reset: `" (dbDelMultiple (sdict "pattern" "dailyClaimed") 100 0) "`")}}
```

## Claim Daily Pulls Command
Trigger Type: `Regex`
Trigger: `\Ag!claim|g!daily|g!claimdaily\z`

Response:
```go
{{$nextReset := (print "<t:" (newDate currentTime.Year currentTime.Month (add currentTime.Day 1) 0 0 0).Unix ":R>")}}
{{if not (dbGet .User.ID "dailyClaimed")}}
    {{$resp := (print "Claimed your daily pulls! You now have `" (dbIncr .user.ID "pulls" 3) "` Pulls available!")}}
    {{sendMessage nil $resp}}
    {{dbSet .User.ID "dailyClaimed" 1}}
{{else}}
    {{$resp := (print "You already claimed your daily! Dailies will reset again " $nextReset)}}
    {{sendMessage nil $resp}}
{{end}}
```

## View available Pulls Command
Trigger Type: `Regex`
Trigger: `\Ag!pulls\z`

Response:
```go
{{$nextReset := (print "<t:" (newDate currentTime.Year currentTime.Month (add currentTime.Day 1) 0 0 0).Unix ":R>")}}
{{if or (not (dbGet (.User.ID) "pulls")) (eq (toInt ((dbGet .User.ID "pulls").Value)) 0)}}
    {{if not (dbGet .User.ID "dailyClaimed")}}
        {{$resp := "No Pulls available!\n**New Daily Available!** Use `g!claim` to claim it!"}}
        {{sendMessage nil $resp}}
    {{else if (dbGet .User.ID "dailyClaimed")}}
        {{$resp := (print "No Pulls available! Dailies will reset " $nextReset)}}
        {{sendMessage nil $resp}}
    {{end}}
{{else}}
    {{if not (dbGet .User.ID "dailyClaimed")}}
        {{$resp := (print "Available Pulls: `" ((dbGet (.User.ID) "pulls").Value) "`\n**New Daily Available!** Use `g!claim` to claim it!")}}
        {{sendMessage nil $resp}}
    {{else if (dbGet .User.ID "dailyClaimed")}}        
        {{$resp := (print "Available Pulls: `" ((dbGet (.User.ID) "pulls").Value) "`")}}
        {{sendMessage nil $resp}}
    {{end}}
{{end}}
```

## Pull Character Command
Trigger Type: `Regex`
Trigger: `\Ag!pull\z`

Response:
```go
{{$rewards := dbGet 98116114 "characterData"}}

{{$totalWeight := 0}}
{{range $key, $value := $rewards.Value}}
  {{$totalWeight = add $totalWeight $value.weight}}
{{end}}

{{if le $totalWeight 0}}
  {{sendMessage nil "No rewards are configured. Please notify an admin."}}
  {{return}}
{{end}}

{{$pulls := dbGet .User.ID "pulls"}}
{{$hasDaily := not (dbGet .User.ID "dailyClaimed")}}
{{$nextReset := (print "<t:" (newDate currentTime.Year currentTime.Month (add currentTime.Day 1) 0 0 0).Unix ":R>")}}

{{if or (not $pulls) (eq (toInt $pulls.Value) 0)}}
  {{if $hasDaily}}
    {{sendMessage nil "No pulls available! **New daily available!** Use `g!claim` to get your free pulls."}}
  {{else}}
    {{sendMessage nil (print "No pulls available! Dailies will reset " $nextReset)}}
  {{end}}
  {{return}}
{{end}}

{{$rand := div (toFloat (randInt 1000)) 10}}
{{$StrippedRep :=  slice (exec "rep") (add (len .User.Username) 4)}}
{{$rep := toFloat (reFind `\d+` $StrippedRep)}}
{{$buff := or (mult (div $rep 10) 2) (toFloat 0)}}
{{$calc := (toFloat (printf "%.1f" (sub $rand $buff)))}}
{{$chosen := ""}}

{{range $key, $value := $rewards.Value}}
  {{$calc = sub $calc $value.weight}}
  {{if le $calc (toFloat 0)}}
    {{$chosen = $value}}
    {{break}}
  {{end}}
{{end}}

{{if (dbGet .User.ID "pullCooldown")}}
  {{sendMessage nil (print "On cooldown for `" (humanizeDurationSeconds ((dbGet .User.ID "pullCooldown").ExpiresAt.Sub currentTime)) "`")}}
{{else if $chosen}}
  {{sendMessage nil (print "Used 1 Pull! Pulls left: `" (dbIncr .User.ID "pulls" -1) "`")}}
  {{$embed := cembed
    "title" (print $chosen.rarity " - " $chosen.name)
    "description" (print "You pulled " $chosen.name "!")
    "image" (sdict "url" $chosen.url)
    "color" $chosen.color
  }}
  {{sendMessage nil $embed}}
  {{sendMessage nil (print "You now have `" (dbIncr .User.ID (lower $chosen.name) 1) "` " $chosen.name "'s in your collection! Use `g!collection` to view it.")}}
  {{dbSetExpire .User.ID "pullCooldown" 1 3}}
{{else}}
  {{sendMessage nil "An error occurred while pulling. Please try again later."}}
{{end}}
```

## View Collection Command
Trigger Type: `Regex`
Trigger: `\Ag!collection`

Response:
```go
{{$args := parseArgs 0 "" (carg "user" "User")}}
{{$user := or ($args.Get 0) .User}}
{{$characterData := (dbGet 98116114 "characterData").Value}}

{{$fields := cslice}}
{{$total := 0}}

{{$collection := cslice}}
{{range $key, $char := $characterData}}
  {{$amount := or (toInt (dbGet $user.ID $key).Value) 0}}
  {{if gt $amount 0}}
    {{$collection = $collection.Append (dict
      "name" $char.name
      "rarity" $char.rarity
      "weight" $char.weight
      "amount" $amount
    )}}
    {{$total = add $total $amount}}
  {{end}}
{{end}}

{{if gt $total 0}}

  {{range $i, $el := $collection}}
    {{range $j, $el2 := $collection}}
      {{if and (gt (index $el "weight") (index $el2 "weight")) (lt $i $j)}}
        {{$tmp := index $collection $i}}
        {{$collection.Set $i (index $collection $j)}}
        {{$collection.Set $j $tmp}}
      {{end}}
    {{end}}
  {{end}}

  {{range $collection}}
    {{$fields = $fields.Append (sdict
      "name" " "
      "value" (print "-# " .rarity "\n**" .name "** (" .amount ")")
      "inline" true
    )}}
  {{end}}
  {{sendMessage nil (cembed
    "title" (print $user.Globalname "'s Collection")
    "description" (print "Total Characters Collected: " $total "")
    "color" 8008351
    "fields" $fields
    "thumbnail" (sdict "url" ($user.AvatarURL "512"))
  )}}
{{else}}
  {{sendMessage nil (print $user.Globalname "'s collection is empty.")}}
{{end}}
```

## Double Pulls or Nothing Command
Trigger Type: `Regex`
Trigger: `\Ag!double\z`

Response:
```go
{{if or (not (dbGet (.User.ID) "pulls")) (eq (toInt ((dbGet .User.ID "pulls").Value)) 0)}}
	{{sendMessage nil "No Pulls to double!"}}
{{else}}
	{{$pulls := (dbGet .User.ID "pulls").Value}}
	{{$rand := randInt 100}}
	{{if le $rand 50}}
		{{sendMessage nil (print "You won the double or nothing! You now got: `" (dbIncr .User.ID "pulls" $pulls) "` pulls." )}}
	{{else}}
		{{dbSet .User.ID "pulls" 0}}
		{{sendMessage nil (print "You lost the double or nothing! You now got `0` pulls." )}}
	{{end}}
{{end}}
```

## Transfer Pulls Command
Trigger Type: `Regex`
Trigger: `\Ag!transfer`

Response:
```go
{{$args := parseArgs 2 ""
  (carg "int" "Pulls" 1 10)
  (carg "user" "User")
 }}
{{$points := toInt ($args.Get 0)}}
{{$target := userArg ($args.Get 1)}}
{{$pulls := toInt ((dbGet (.User.ID) "pulls").Value)}}
{{$pullsDel := (print "-" ($args.Get 0))}}

{{if (targetHasRole $target.ID 1279084386398244915)}}
{{sendMessage nil "Can't transfer pulls to bots!"}}
{{else if lt $pulls $points}}
{{sendMessage nil (print "**Not enough pulls for his transfer!**\nYou only got `" $pulls "` Pulls!")}}
{{else}}
{{sendMessage nil (print "Transfer successful, gave `" $points "` Pulls to **" $target.Globalname "**! (Now has `" (dbIncr $target.ID "pulls" $points) "` pulls total)\nYou now have `" (dbIncr .User.ID "pulls" $pullsDel) "` pulls left!")}}
{{end}}
```

## Exchange YAGPDB Reputation for Pulls Command (Optional)
Trigger Type: `Regex`
Trigger: `\Ag!exchange`

Response:
```go
{{$args := parseArgs 1 "" (carg "int" "Points" 1 10)}}
{{$points := toInt ($args.Get 0)}}

{{$StrippedRep :=  slice (exec "rep") (add (len .User.Username) 4)}}
{{$rep := toInt (reFind `\d+` $StrippedRep)}}

{{if lt $rep $points}}
  {{sendMessage nil (print "**Not enough points for his exchange!**\nYou only got `" $rep "` Freak Points!\n-# Conversion Rate: `1:1`")}}
{{else}}
  {{$newrep := sub $rep $points}}
  {{execAdmin "setrep" .User.ID $newrep}}
  {{sendMessage nil (print "Exchange successful, converted `" $points "` Freak Points to Pulls! You now have `" (dbIncr .User.ID "pulls" $points) "` Pulls total!")}}
{{end}}
```

## Gacha Commands Help Command
Trigger Type: `Regex`
Trigger: `\A(g!help|g!h|g!)\z`

Response:
```go
## BtR! Gacha Command Help:
`g!claim` - Claim your daily pull!

`g!pulls` - View how many pulls you currently have!

`g!pull` - Pull for a BtR! Character!

`g!chances` - View pull chances!

`g!collection` - View your collection of pulled characters!

`g!collection <Mention/UserID>` - View other people's collection!

`g!double` - Double or nothing! Will you lose all your pulls or double them?

`g!exchange` - Exchange your Freak Points for extra pulls!

`g!transfer <Pulls> <Target User>` - Transfer pulls to other players!

`g!trade help` - View help on player trading commands

`g!help` - View this help message
{{deleteTrigger 15}}
{{deleteResponse 15}}
```

# Trade Commands

## Create Trade Offer Command
Trigger Type: `Regex`
Trigger: `\Ag!trade offer`

Response:
```go
{{$args := parseArgs 4 ""
  (carg "user" "Target User")
  (carg "int" "Amount" 1)
  (carg "string" "Character")
  (carg "int" "Price (Pulls)" 1)
 }}

{{$target := $args.Get 0}}
{{$amount := toInt ($args.Get 1)}}
{{$char := lower ($args.Get 2)}}
{{$price := toInt ($args.Get 3)}}
{{$targetPulls := toInt ((dbGet ($target.ID) "pulls").Value)}}
{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$validChars := cslice}}

{{range $key, $_ := $characterData}}
  {{$validChars = $validChars.Append (lower $key)}}
{{end}}

{{if .User.Bot}}
	{{sendMessage nil "Can't trade with bots!"}}
{{else if lt $targetPulls $price}}
	{{sendMessage nil (print "**" $target.Globalname "** doesn't have enough pulls for this trade!")}}
{{else if not (in $validChars $char)}}
	{{sendMessage nil "Invalid Character entered!\n-# Check `g!chars` for valid Characters"}}
{{else if not (dbGet .User.ID $char)}}
	{{sendMessage nil (print "You don't have any " $char "'s!")}}
{{else if lt (toInt ((dbGet .User.ID $char).Value)) $amount}}
	{{sendMessage nil (print "You don't have enough " $char "'s for this trade offer!")}}
{{else if (dbGet $target.ID "tradeOffer")}}
	{{sendMessage nil (print $target.Globalname " already has a trade offer pending!")}}
{{else}}
	{{$nonLowerChar := index $characterData $char}}
	{{$charName := (index $nonLowerChar "name")}}
	{{sendMessage nil (print $target.Mention ", you just got a trade offer from " .User.Mention "!\nThey want to trade `" $amount "` " $charName "'s with you in exchange for `" $price "` pulls!\n:warning: **Offer will expire <t:" (currentTime.Add (toDuration "10m")).Unix ":R>**\n-# Use `g!trade accept` to accept or `g!trade decline` to decline the offer")}}
  {{sendMessage nil "https://tenor.com/view/trade-offer-gif-22327121"}}
  {{dbSetExpire $target.ID "tradeOffer" 1 600}}
	{{dbSetExpire $target.ID "tradeChar" $char 600}}
	{{dbSetExpire $target.ID "tradeAmount" $amount 600}}
	{{dbSetExpire $target.ID "tradePrice" $price 600}}
	{{dbSetExpire $target.ID "tradeInitiator" .User.ID 600}}
{{end}}
```

## Accept Trade Offer Command
Trigger Type: `Regex`
Trigger: `\Ag!trade accept\z`

Response:
```go
{{$char := toString (dbGet .User.ID "tradeChar").Value}}
{{$amount := toInt (dbGet .User.ID "tradeAmount").Value}}
{{$price := toInt (dbGet .User.ID "tradePrice").Value}}
{{$initiator := userArg (toInt (or (dbGet .User.ID "tradeInitiator").Value .User.ID))}}
{{$initiatorChars := toInt (dbGet $initiator.ID $char).Value}}
{{$targetPulls := toInt (dbGet .User.ID "pulls").Value}}
{{$characterData := (dbGet 98116114 "characterData").Value}}
{{$charsDel := (print "-" $amount)}}
{{$pullsDel := (print "-" $price)}}

{{if not (dbGet .User.ID "tradeOffer")}}
	{{sendMessage nil "You currently have no trade offers!"}}
{{else if lt $targetPulls $price}}
	{{sendMessage nil "Not enough pulls for trade offer! Deleting offer..."}}
	{{dbDel .User.ID "tradeOffer"}}
	{{dbDel .User.ID "tradeChar"}}
	{{dbDel .User.ID "tradeAmount"}}
	{{dbDel .User.ID "tradePrice"}}
	{{dbDel .User.ID "tradeInitiator"}}
{{else if lt $initiatorChars $amount}}
	{{sendMessage nil (print "**" $initiator.Globalname "** doesn't have enough" $char "'s for this trade anymore! Deleting offer...")}}
{{else if (dbGet .User.ID "tradeOffer")}}
	{{$nonLowerChar := index $characterData $char}}
	{{$charName := (index $nonLowerChar "name")}}
	{{sendMessage nil (print "Transfer successful, gave `" $amount "` " $charName "'s to **" .User.Globalname "** for `" $price "` pulls! You now have `" (dbIncr .User.ID "pulls" $pullsDel) "` pulls and `" (dbIncr .User.ID $char $amount) "` " $charName "'s\n" $initiator.Globalname " now has `" (dbIncr $initiator.ID "pulls" $price) "` pulls and has `" (dbIncr $initiator.ID $char $charsDel) "` " $charName "'s left!")}}
	{{dbDel .User.ID "tradeOffer"}}
	{{dbDel .User.ID "tradeChar"}}
	{{dbDel .User.ID "tradeAmount"}}
	{{dbDel .User.ID "tradePrice"}}
	{{dbDel .User.ID "tradeInitiator"}}
{{end}}
```

## Decline Trade Offer Command
Trigger Type: `Regex`
Trigger: `\Ag!trade decline\z`

Response:
```go
{{if not (dbGet .User.ID "tradeOffer")}}
{{sendMessage nil "You currently have no trade offers!"}}
{{else if (dbGet .User.ID "tradeOffer")}}
{{sendMessage nil "Offer declined!"}}
{{dbDel .User.ID "tradeOffer"}}
{{dbDel .User.ID "tradeChar"}}
{{dbDel .User.ID "tradeAmount"}}
{{dbDel .User.ID "tradePrice"}}
{{dbDel .User.ID "tradeInitiator"}}
{{end}}
```

## Valid Characters Command
Trigger Type: `Regex`
Trigger: `\Ag!chars\z`

Response:
```go
{{$rewards := (dbGet 98116114 "characterData").Value}}

{{$response := "**Valid Characters:**\n\n"}}
{{range $rewards}}
  {{$response = print $response "`" .name "`    "}}
{{end}}

{{$message := sendMessageRetID nil $response}}
{{deleteTrigger 15}}
{{deleteMessage nil $message 15}}
```

## Trade Commands Help Command
Trigger Type: `Regex`
Trigger: `\A(g!trade help|g!trade)\z`

Response:
```go
## BtR! Gacha Trading Commands:
`g!trade offer <Target User> <Amount> <Character> <Price (Pulls)>` - Offer a trade to another user!

`g!trade accept` - Accept someone's trade!

`g!trade decline` - Decline someone's trade!
{{deleteTrigger 15}}
{{deleteResponse 15}}
```
