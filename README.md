# nng-haskell

This is a Haskell binding for the NNG library: <http://nng.nanomsg.org/>.

It's based on the nanomsg bindings: https://github.com/bgamari/nanomsg-haskell.

This bindings make use of the "compatibility API" from NNG to emulate the same API
nanomsg has. Thus at the moment it doesn't offer the full functionality of NNG,
but it should be easy to extend over time as it's needed.

There's support for [(evented)](http://hackage.haskell.org/packages/archive/base/latest/doc/html/Control-Concurrent.html#v:threadWaitRead) blocking send and recv, a non-blocking receive,
and for all the socket types and the functions you need to wire them up and
tear them down again.

Most socket options are available through accessor and mutator
functions. Sockets are typed, transports are not.


## Building

You would normally make sure the NNG library is on your system and then
install from Hackage.


## Usage

Simple pub/sub example:

Server:
```haskell
module Main where

import Nanomsg
import qualified Data.ByteString.Char8 as C
import Control.Monad (mapM_)
import Control.Concurrent (threadDelay)

main :: IO ()
main =
    withSocket Pub $ \s -> do
        _ <- bind s "tcp://*:5560"
        mapM_ (\num -> sendNumber s num) (cycle [1..1000000 :: Int])
    where
        sendNumber s number = do
            threadDelay 1000        -- let's conserve some cycles
            let numAsString = show number
            send s (C.pack numAsString)
```

Client:
```haskell
module Main where

import Nanomsg
import qualified Data.ByteString.Char8 as C
import Control.Monad (forever)

main :: IO ()
main =
    withSocket Sub $ \s -> do
        _ <- connect s "tcp://localhost:5560"
        subscribe s $ C.pack ""
        forever $ do
            msg <- recv s
            C.putStrLn msg
```

Nonblocking client:
```haskell
module Main where

import Nanomsg
import qualified Data.ByteString.Char8 as C
import Control.Monad (forever)
import Control.Concurrent (threadDelay)

main :: IO ()
main =
    withSocket Sub $ \s -> do
        _ <- connect s "tcp://localhost:5560"
        subscribe s $ C.pack ""
        forever $ do
            threadDelay 700           -- let's conserve some cycles
            msg <- recv' s
            C.putStrLn $ case msg of
                Nothing -> C.pack "No message"
                Just m  -> m
```
