+++
author = "Jefferson Rocha"
title = "#007 | Realtime communication with SignalR"
date = 2023-01-27T06:12:03Z
description = ""
subtitle = "Ever wondered how realtime applications are implemented?"
header_img = ""
short = false
toc = true
tags = ["Fluent validation", "C#", "dotnet"]
categories = []
series = []
comment = true
+++

# SignalR

SignalR is .Net's solution to realtime communications. It enables the creation of hubs and then efficiently route all messages to all clients.

You can checkout this [Hi-Lo Game](https://hi-lo-game.azurewebsites.net) which leverages SignalR ([source](https://github.com/JeffersonAmori/HighLowGame)).

![state_machine](../../../images/Posts/007/Hi-Lo-Game.png)

The idea is that all messages from all player (and also the game engine) are broadcasted to all players (connect to.

# The implementation

## Server side

[GameHub.cs](https://github.com/JeffersonAmori/HighLowGame/blob/master/HighLowGame/Hubs/GameHub.cs)

A cluster of connections is called a `hub`. The two main methods of our `hub` are the `Guess` and `WriteToPageAsync` methods:

```csharp
    /// <summary>
    /// Called from the client when a player guesses the Mystery Number.
    /// </summary>
    /// <param name="user">Player's name</param>
    /// <param name="guess">Player's guess</param>
    /// <returns></returns>
    public async Task Guess(string user, string guess)
    {
        _logger.LogInformation("{user} guessed {guess}", user, guess);
        if (!int.TryParse(guess, out int guessedNumber))
        {
            var errorMessage = $"Your guess {guess} should be a number (ex: 10).";
            _logger.LogError(errorMessage);
            await WriteToPageAsync(user, errorMessage);
            return;
        }

        await WriteToPageAsync(user, $"{user} guesses {guess}.");
        var responseToGuess = _gameMaster.ValidateGuess(user, guessedNumber);
        await WriteToPageAsync(GameMasterUser, MessageFrom(responseToGuess));

        if (responseToGuess.Value == ResponseToGuess.Correct)
        {
            await CelebrateAsync(user);
            await PrintStatisticsAsync();
            await StartNewRoundAsync();
        }
    }

    private async Task WriteToPageAsync(string user, string message)
    {
        _logger.LogInformation("Writing message {message} from {user}", message, user);
        await Clients.All.SendAsync("WriteToPage", user, message);
    }
```

They are, respectively, the entry and exit points for the `GameHub`. 
The clients call `Guess` to validade the player's guess and `GameHub` uses `WriteToPageAsync` to communicate back with the clients
(note the `await Clients.All.SendAsync("WriteToPage", user, message);` call).

