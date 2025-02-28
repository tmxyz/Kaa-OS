## Rustilla kirjoitetun käyttöjärjestelmän dokumentointi

Tämä on oma, vapaamuotoinen dokumentaatio käyttöjärjestelmän kehittämisestä Rustilla Phil Oppin loistavan blogin pohjalta: [os.phil-opp.com](https://os.phil-opp.com/). Projektia rakennetaan **rustc nightly** -versiolla. Tarkemmat ohjeet löytyvät alkuperäisestä tutoriaalista.

### Projektin alustus

**Cargo.toml**
- Määritetään `edition = "2018"`.

**src/main.rs**
- Poistetaan `std`-kirjasto ja perinteinen `main`-funktio, jotta Rust ei linkitä ohjelmaa ulkoisiin kirjastoihin.
- Lisätään **panic-handler** ja ohjelman aloituskohta `_start()`.
- Käytetään `#[no_mangle]`, jotta kääntäjä ei muunna `_start`-funktion nimeä.

**Cargo.toml**
- Lisätään panic-käsittely profiileihin (`profile.dev` ja `profile.release`):
  - Jos ohjelma panikoi (`panic!`), suoritetaan `abort` ilman lisäkäsittelyä.

### Kääntäjän asetukset

- Luodaan kääntäjälle asetustiedosto:
  ```sh
  touch x86_64-kaa_os.json
  ```
  Tämä määrittää mm. kohdejärjestelmän ja `panic-strategy: "abort"`.

- Luodaan `.cargo/`-hakemisto ja sinne `config.toml`:
  ```sh
  mkdir .cargo && touch .cargo/config.toml
  ```
  - Jos **nightly-versiota** ei ole käytössä, projekti ei toimi.
  - Määritellään `compiler-builtins-mem`, joka korvaa C-kieliset muistinhallintafunktiot.
  - Asetetaan oletuskohde `.json`-tiedostolle.

### Ensimmäinen ohjelma

Kirjoitetaan **"Hello, World!"** -tulostus käyttämällä **VGA text bufferia**. Tässä vaiheessa `_start()`-funktioon lisätään ensimmäiset suorituskäskyt. Virheitä kannattaa välttää!

### Bootloaderin lisääminen

**Cargo.toml**
- Lisätään riippuvuus:
  ```toml
  [dependencies]
  bootloader = "x.x.x"  # Tarkista uusin versio
  ```
  Bootloader alustaa prosessorin ja lataa käyttöjärjestelmän muistiin.

- Asennetaan `bootimage`:
  ```sh
  cargo install bootimage
  ```
  Tämä vaatii **LLVM-työkalut**:
  ```sh
  rustup component add llvm-tools-preview
  ```

### Käynnistys Qemu-emulaattorissa

Voimme luoda boot imagen ja ajaa sen USB-tikulta, mutta helpompi tapa on käyttää **Qemu-emulaattoria**:

1. Asennetaan Qemu (Linuxilla):
   ```sh
   sudo apt-get install qemu-system
   ```
2. Lisätään `.cargo/config.toml`-tiedostoon:
   ```toml
   [target.'cfg(target_os = "none")']
   runner = "bootimage runner"
   ```
3. Käynnistetään käyttöjärjestelmä:
   ```sh
   cargo run
   ```

Nyt Qemu käynnistyy ja suorittaa kirjoitetun käyttöjärjestelmän.


