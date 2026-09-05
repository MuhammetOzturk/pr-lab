# PR Egzersiz Rehberi (pr-lab)

Kendi başına, sıfır riskle (özel depo) PR yaşam döngüsünü denemek için egzersizler.
Her egzersizden önce `git checkout main && git pull` ile temiz başla.

## D1 — İlk PR (temel akış)

```bash
git checkout -b deneme-1
echo "satır 1: deneme" >> README.md
git commit -am "deneme değişikliği"
git push -u origin deneme-1
gh pr create --fill          # başlık/gövdeyi commit'ten doldurur
gh pr view                   # PR'i incele
gh pr comment --body "kendi yorumum"
gh pr merge --squash --delete-branch   # main'e birleştir + branch temizliği
```

**Gözle:** `gh pr merge` sonrası local + remote branch silinir; README main'e geçer.

## D2 — Issue bağlantısı ve otomatik kapanma

```bash
gh issue create -t "Egzersiz issue" -b "Bu issue D2'de kapanacak"
git checkout -b fix-2
echo "satır 2: fix" >> README.md && git commit -am "fix" && git push -u origin fix-2
gh pr create --fill-first --body "Fixes #<issue-numarası>"
gh pr merge --squash --delete-branch
gh issue list --state closed --limit 1   # issue otomatik kapandı mı?
```

**Kural:** Gövdede `Fixes #N` / `Closes #N` → merge'te issue kapanır.

## D3 — Taslak (draft) PR

```bash
git checkout -b taslak
echo "yarım iş" >> README.md && git commit -am "wip" && git push -u origin taslak
gh pr create --draft --fill
gh pr ready                  # review'e hazır hale getir
gh pr close --delete-branch
```

## D4 — Merge conflict üretme ve çözme

```bash
# main'de doğrudan değişiklik (branch'siz)
git checkout main
echo "satır A" >> README.md && git commit -am "main'e A" && git push

# branch'te AYNI satıra farklı değişiklik
git checkout -b cakisan && sed -i 's/satır A/satır B/' README.md
git commit -am "branch'te B" && git push
gh pr create --fill-first
gh pr merge --squash   # → conflict hatası gör
```

Çözüm:

```bash
git checkout cakisan && git pull origin main   # veya: gh pr update-branch
# conflict'i elle çöz → git add → git commit
git push && gh pr merge --squash --delete-branch
```

## D5 — Kendi PR'ını inceleyememe kuralı

```bash
gh pr review --comment --body "not: kendi PR'ıma yorum yapabiliyorum"
gh pr review --approve   # → GitHub HATASI döndürür: "Can not approve your own pull request"
```

**Öğrenilen:** PR yazarı kendi PR'ını onaylayamaz (gerçek ekipte 2. göz zorunluluğunun sebebi).
Kendi kendine çalışma: `--comment` kullan, veya ikinci bir hesapla dene.

## D6 — Merge stratejilerinin geçmişteki izi

Aynı egzersizi üç kez yap, her seferinde farklı strateji kullan:
`--merge`, `--squash`, `--rebase`. Sonra karşılaştır:

```bash
git checkout main && git pull
git log --oneline --graph -10     # her stratejinin çıktısı farklıdır
```

- `--merge`: branch commit'leri + merge commit
- `--squash`: branch'in tüm commit'leri tek commit'e düşer
- `--rebase`: commit'ler main üzerine tek tek yeniden yazılır

## D7 — CI check simülasyonu (workflow scope gerekir)

Token ayarlarında **Workflows** iznini açtıktan sonra:

```bash
mkdir -p .github/workflows
cat > .github/workflows/ci.yml <<'EOF'
name: CI
on: [push, pull_request]
jobs:
  deneme:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "her şey yolunda" && exit 0
EOF
git checkout -b ci && git add . && git commit -m "ci ekle" && git push -u origin ci
gh pr create --fill
gh pr checks --watch          # yeşile kadar bekle
gh run list --limit 1         # Actions çıktısını gör
```

Kasıt testi: `exit 0` yerine `exit 1` yaz → `gh pr checks` kırmızı döner →
`gh pr merge --auto --squash` ver (merge beklemede kalır) → commit'i düzelt, auto-merge kendisi devreye girer.

## D8 — PR'i local'e almak

```bash
git checkout main
gh pr checkout <numara>       # PR'in branch'i local'e gelir
gh pr diff <numara>           # diff'i terminalde gör
gh pr view --web              # tarayıcıda aç, GUI tarafını da tanı
```

## Temizlik

Deneme branch'leri birikirse:

```bash
gh pr list --state merged
git branch -d deneme-1 taslak ...   # local temizlik (merge edilmişler -d ile gider)
```

Depoyu tamamen sıfırlamak istersen: `gh repo delete MuhammetOzturk/pr-lab --yes`
(delete_repo izni gerekebilir) ve yeniden `gh repo create pr-lab --private`.