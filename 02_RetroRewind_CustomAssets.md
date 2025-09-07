# Retro RewindでカスタムUIAssetsが読み込めなくなったときの対処法
最終更新日: 2025年9月7日

## この記事の対象者
- Retro Rewindでアセット編集を行いたい方
- 内部情報を知りたい方
- テクスチャハックの基礎的な知識を有している方

## 概要
Pulsar用に追加されたファイルが格納される`UIAssets.szs`を編集して、My Stuff経由で読み込もうとしたが正常に反映されなかった。

## 解決策
- `UIAssets_J.szs`にリネームをする。

## ファイル構成について
解決のために、Retro RewindのソースコードとパックのXMLファイルを覗いた。\
`Misc.cpp`ファイルは以下のようになっている。
```cpp
#include <kamek.hpp>
#include <MarioKartWii/Archive/ArchiveMgr.hpp>
#include <MarioKartWii/Scene/GameScene.hpp>
#include <Settings/Settings.hpp>
#include <PulsarSystem.hpp>

namespace Pulsar {

// Adds a 3rd UI (menu or race) and a 3rd common to the relevant archive holders, which will contain custom pulsar assets
kmWrite32(0x8052a108, 0x38800003);  // Add one archive to CommonArchiveHolder
kmWrite32(0x8052a188, 0x38800004);  // Add one archive to UIArchiveHolder
void LoadAssetsFile(ArchiveFile* file, const char* path, EGG::Heap* decompressedHeap, bool isCompressed, s32 allocDirection,
                    EGG::Heap* archiveHeap, EGG::Archive::FileInfo* info) {
    const ArchiveMgr* archiveMgr = ArchiveMgr::sInstance;
    if (file == &archiveMgr->archivesHolders[ARCHIVE_HOLDER_UI]->archives[3]) {
        const char* fileType = "UI";
        Pulsar::Language currentLanguage = static_cast<Pulsar::Language>(Pulsar::Settings::Mgr::Get().GetUserSettingValue(static_cast<Pulsar::Settings::UserType>(Pulsar::Settings::SETTINGSTYPE_RRLANGUAGE), Pulsar::SETTINGRRLANGUAGE_LANGUAGE));

        bool isRaceScene = (GameScene::GetCurrent()->id == SCENE_ID_RACE);
        const char* baseType = isRaceScene ? "Race" : "UI";
        const char* langSuffix = "";
        switch (currentLanguage) {
            case Pulsar::LANGUAGE_JAPANESE:
                langSuffix = "_J";
                break;
            case Pulsar::LANGUAGE_FRENCH:
                langSuffix = "_F";
                break;
            case Pulsar::LANGUAGE_GERMAN:
                langSuffix = "_G";
                break;
            case Pulsar::LANGUAGE_DUTCH:
                langSuffix = "_D";
                break;
            case Pulsar::LANGUAGE_SPANISHUS:
                langSuffix = "_AS";
                break;
            case Pulsar::LANGUAGE_SPANISHEU:
                langSuffix = "_ES";
                break;
            case Pulsar::LANGUAGE_FINNISH:
                langSuffix = "_FI";
                break;
            case Pulsar::LANGUAGE_ITALIAN:
                langSuffix = "_I";
                break;
            case Pulsar::LANGUAGE_KOREAN:
                langSuffix = "_K";
                break;
            case Pulsar::LANGUAGE_RUSSIAN:
                langSuffix = "_R";
                break;
            case Pulsar::LANGUAGE_TURKISH:
                langSuffix = "_T";
                break;
            case Pulsar::LANGUAGE_CZECH:
                langSuffix = "_C";
                break;
            case Pulsar::LANGUAGE_ENGLISH:
                langSuffix = "";
                break;
            default:
                langSuffix = "";
                break;
        }
        char newPath[0x20];
        snprintf(newPath, 0x20, "%sAssets%s.szs", baseType, langSuffix);
        path = newPath;
    } else if (file == &archiveMgr->archivesHolders[ARCHIVE_HOLDER_COMMON]->archives[2])
        path = System::CommonAssets;
    else if (file == &archiveMgr->archivesHolders[ARCHIVE_HOLDER_UI]->archives[0])
        path = "/ReplacedAssets.szs";
    file->Load(path, decompressedHeap, isCompressed, allocDirection, archiveHeap, info);
}
kmCall(0x8052aa2c, LoadAssetsFile);

}  // namespace Pulsar
```

ゲームシーンと言語設定に応じて拡張アセットファイルを読み込むためのコードである。\
例えば**レースシーン**において言語設定が**ロシア語**の場合、読み込まれるファイルは`RaceAssets_R.szs`である。また、xmlファイルは以下のようになっていた。
```xml
...
<file disc="/UIAssets_J.szs" external="/RetroRewind6/Language/JPN/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_F.szs" external="/RetroRewind6/Language/FRA/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_G.szs" external="/RetroRewind6/Language/GER/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_D.szs" external="/RetroRewind6/Language/DUT/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_AS.szs" external="/RetroRewind6/Language/SPA(NTSC)/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_ES.szs" external="/RetroRewind6/Language/SPA(EU)/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_FI.szs" external="/RetroRewind6/Language/FIN/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_I.szs" external="/RetroRewind6/Language/ITA/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_K.szs" external="/RetroRewind6/Language/KOR/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_R.szs" external="/RetroRewind6/Language/RUS/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_T.szs" external="/RetroRewind6/Language/TUR/Assets/UIAssets.szs" create="true"/>
<file disc="/UIAssets_C.szs" external="/RetroRewind6/Language/CZE/Assets/UIAssets.szs" create="true"/>
<file disc="/RaceAssets_J.szs" external="/RetroRewind6/Language/JPN/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_F.szs" external="/RetroRewind6/Language/FRA/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_G.szs" external="/RetroRewind6/Language/GER/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_D.szs" external="/RetroRewind6/Language/DUT/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_AS.szs" external="/RetroRewind6/Language/SPA(NTSC)/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_ES.szs" external="/RetroRewind6/Language/SPA(EU)/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_FI.szs" external="/RetroRewind6/Language/FIN/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_I.szs" external="/RetroRewind6/Language/ITA/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_K.szs" external="/RetroRewind6/Language/KOR/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_R.szs" external="/RetroRewind6/Language/RUS/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_T.szs" external="/RetroRewind6/Language/TUR/Assets/RaceAssets.szs" create="true"/>
<file disc="/RaceAssets_C.szs" external="/RetroRewind6/Language/CZE/Assets/RaceAssets.szs" create="true"/>
...
```

この2つのファイルから、ディスクの構造としては`〇〇Assets_〇.szs`として存在しているものを、パックの`Language`フォルダに含まれる`UIAssets.szs`で置き換えていることが分かった。\
したがってMy Stuffに入れて使用する際は`UIAssets.szs`ではなく、ディスク構造に合致するように`UIAssets_J.szs`として読み込む必要があるといえる。

## おわりに
今回はカスタム`UIAssets`を使用し続ける方法についてまとめました。日本語を使用する場合は`_J`をつけますが、その他の言語で反映させたい場合は適宜リネームして対応してください。なお、`RaceAssets`でも同様に対応して反映させることができます。