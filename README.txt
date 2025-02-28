This is my own loose documentation about following the process of writing an operating system with Rust in this great blog: https://os.phil-opp.com/

Projektia rakennellaan rustc nightly versiolla.
Tämä on oma löyhä dokumentointi tehdyistä toimenpiteistä. Tarkkoja ohjeita varten
suosittelen lukemaan ja seuraamaan alkuperäistä tutoriaalia yllä mainitussa linkissä.

Tutoriaalin mukaisesti merkataan Cargo.toml >> edition 2018.

src/main.rs >> poistetaan käytöstä std-kirjasto ja perinteinen main-kutsu,
koska emme halua, että rust linkittää ohjelmaa mihinkään ulkoiseen kirjastoon,
minkä se siis tekee oletuksena. Lisäksi tehdään samaan tiedostoon tarpeelliset
toimenpiteet, eli panic-handler ja ohjelman aloituskohta, eli _start()-funktio.

#[no_mangle]-atribuutti kertoo kääntäjälle olla tekemättä omaa nimeä funktiolle.
Oletuksena kääntäjä antaa funktioille omat uniikit kjda+d8i32ijr3kr-nimet muistissa.

Cargo.toml >> lisätään panic-ohjeet profiileihin profile.dev ja profile.release.
Eli mikäli ohjelma (Rust) panikoi (panic!), eli kohtaa virheen, se suorittaa komennon
"abort", eikä mitään muuta.

Luodaan .json-asetustiedosto projektin juureen >> "touch x86_64-kaa_os.json".
Tämä sisältää ohjeita kääntäjälle, mm. target systemin, ynnä muuta hauskaa ja mielenkiintoista, kuten edellä jo tehdyn panic-handler ohjeen "panic-strategy": "abort", minkä voimme nyt
joko poistaa Cargo.toml-tiedostosta tai jättää sinne.

Huom! KAIKKI toimenpiteet tämän kaltaisen ohjelman rakentamisessa ovat tarkkoja ja tärkeitä.
Jonkin ohjeen tai asetuksen unohtaminen, tai virhe ohjelmassa voi johtaa ikäviin seuraamuksiin!

luodaan .cargo/ -kansio projektin juureen ja sinne config.toml-niminen tiedosto. Jos nyt et käytä
rust nightly-versiota niin ei toimi. Ellei jo aikaisemmin tullut ongelmia.

.cargo/config.toml >> lisätään ohjeita cargolle projektin rakentamiseen liittyen ja nämä ovat nyt
niitä unstable-ominaisuuksia, joita on pakko käyttää, kun rakennetaan raudalle. Cargon täytyy
rakentaa mm erillinen std-kirjasto, koska ohjelmaa ei voida linkittää olemassaolevaan sellaiseen. Tarkoitus on rakentaa selfstanding ohjelmisto ajettavaksi suoraan BIOSista. UEFI-tukea ei
kuulemma vielä ole, tai ollut tutoriaalin kirjoittamisen aikaan, joten mekään emme sellaista
läde tässä yrittämään.

.cargo/config.toml >> "compiler-builtins-mem" korvaa C:n "memset" ynnä muut muistin varaamiseen ja
hallintaan liittyvät toiminnot Rustin omilla. Näitä ominaisuuksia on pakko joka ohjelman
sisältää ja yleensä ne on toteutettu C:llä.

.cargo/config.toml >> lisätään myös default target ohjaus .json-tiedostoomme

Sitten kirjoitellaan "Helo World" tulostus src/main.rs:ään. Tässä tulee mielenkiintoista
settiä, kun käytetään VGA text bufferia. Eli _start()-funktioon tulee nyt sisältöä, ihan
oikeita suorituskäskyjä. Tässä vaiheessa ei ainakaan kannata tehdä virheitä!

BOOTIMAGE
Lisätään Cargo.toml tiedostoon riippuvuus "bootloader". Tämän joku mestari voisi kirjoittaa
itse, mutta ohjeessa onneksi käytetään valmista pakettia, jonka rust lataa cargolla ohjelmaan.
Bootloader mm. alustaa prosessorin ja lataa ohjelman (kernelin) koneen suoritinmuistiin.

Cargo ei osaa ohjelman kääntämisen jälkeen linkittää sitä bootloaderiin, joten joudumme
käyttämään tähän valmista ominaisuutta ja lisäämme sen komennolla "cargo install bootimage"

bootimage ei toimi ilman työkaluja, jotka saa käyttöön komennolla:
"rustup component add llvm-tools-preview"

NYT voimme luoda suoritettavan boot imagen, laittaa sen vaikka usb-tikulle ja käynnistää
ohjelman koneen boot-menusta jos tikku on kiinnitettynä porttiin. En ole kokeillut.

Tämän sijaan käytämme Linuxilla Qemu-nimistä ohjelmaa. Se on virtuaalikone, eli tietokone-
emulaattori, jolla voimme testata käyttöjärjestelmämme toimivuutta.

"sudo apt-get install qemu-system"

Kun Qemu on asennettu, komennolla:
qemu-system-x86_64 -drive format=raw,file=target/x86_64-blog_os/debug/bootimage-blog_os.bin
ohjelman pitäisi toimia, mutta itselläni ei toiminut!

Itselläni ohjelma toimi, kun lisäsin .cargo/config.toml-tiedostoon rivit:
[target.'cfg(target_os = "none")']
runner = "bootimage runner"
, tämän jälkeen yksinkertainen "cargo run"-komento käynnistää Qemun ja siinä
kirjoittamamme "käyttöjärjestelmän".
