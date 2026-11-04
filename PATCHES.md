# Streamit patches to better_player_plus

This is a public copy of [SunnatilloShavkatov/better_player_plus](https://github.com/SunnatilloShavkatov/better_player_plus)
(Apache-2.0). It exists to carry the patches below until upstream has them. The app uses it as a git dependency
(`streamit-mobile/pubspec.yaml`), pinned to a tag, so no credentials are needed to fetch it.

It is a standalone repository, not a GitHub fork, so there is no "Sync fork" button: new upstream versions are pulled
with git, as described at the end.

Branches and tags:

- `master`: a mirror of upstream `master`. Never commit patches here.
- `streamit/<version>`: upstream tag `<version>` plus our patches, one commit per patch.
- `<version>-streamit.<n>`: a tag on that branch. This is what the app pins. A new tag is made for every change.

## Patches

### 1. Android: a rebuffer is not a pause

File: `android/src/main/kotlin/uz/shs/better_player_plus/BetterPlayer.kt`, `onIsPlayingChanged`.

ExoPlayer reports `isPlaying = false` whenever it rebuffers, although `playWhenReady` is still true. The plugin sent
that to Dart as a `"pause"` event, and the Dart side (`video_player.dart`) answers a `pause` event by really pausing the
player. After the buffer filled nothing resumed it, so a mid-play stall left the video paused with a play icon instead
of a loader.

The patch returns without sending `"pause"` when `!isPlaying && playWhenReady && playbackState == STATE_BUFFERING`.
`bufferingStart` / `bufferingEnd` are still sent from `onPlaybackStateChanged`, and `isPlaying` becoming true again
sends `"play"` as before. Pauses by the user or the media notification (`playWhenReady == false`) are unchanged. iOS
never had this behaviour and is untouched.

Upstream status: not fixed as of 2.0.1 (`onIsPlayingChanged` is unchanged). If upstream fixes it, drop the patch and
point the app back at pub.dev.

## Moving to a new upstream version

```
git remote add upstream https://github.com/SunnatilloShavkatov/better_player_plus.git   # once
git fetch upstream --tags
git checkout -b streamit/<new> <new>                 # branch from the new upstream tag
git cherry-pick <patch commit(s) from streamit/<old>>   # re-apply the patches
# resolve conflicts if upstream touched the same code, then check the patch is still needed
git tag <new>-streamit.1 && git push origin streamit/<new> <new>-streamit.1
git checkout master && git merge --ff-only upstream/master && git push origin master   # keep the mirror current
```

Then change the `ref:` of `better_player_plus` in the app's `pubspec.yaml` to the new tag, run `flutter pub get`, and
build an Android APK to confirm the Kotlin compiles. Check upstream's changelog first: 2.x raised the minimum Dart and
Flutter versions and the iOS target.
