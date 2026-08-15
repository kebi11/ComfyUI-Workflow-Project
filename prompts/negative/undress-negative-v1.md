# undress-negative-v1

状态：已写入 AutoClothes Phase 1，未出图验证。不覆盖 `base-negative.md`。

```text
bad quality, worst quality, worst detail,
clothes, dress, fabric, sheer cloth, transparent cloth,
double clothing, ghost fabric, floating cloth,
tan skin, dark skin, sunburn, uneven skin tone,
wide waist, thick thighs, extra limbs, bad anatomy,
child, loli
```

## 使用说明

- `ghost fabric` / `sheer cloth` 针对原图薄纱残影
- `tan skin` / `uneven skin tone` 针对新皮肤和原腿臂不一致
- `wide waist` / `thick thighs` 只抑制去衣时变壮，不保证锁死身材
- 若负向把身体画没或画残，记入 EXP，不要默默删词
