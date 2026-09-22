<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Hormigón 3D · Simulador de cubicación</title>
<style>
  :root{
    --bg:#e8ecf2; --panel:#ffffff; --line:#d9dfe8; --line-soft:#eaeef4;
    --text:#1f2937; --muted:#6b7280; --accent:#2563eb; --accent-dark:#1d4ed8;
    --ok:#16a34a; --warn:#ea580c; --bad:#dc2626; --gold:#f59e0b;
    --shadow:0 1px 2px rgba(16,24,40,.06), 0 4px 14px rgba(16,24,40,.06);
  }
  *{box-sizing:border-box}
  html,body{height:100%;margin:0}
  body{font-family:ui-sans-serif,system-ui,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
       background:var(--bg);color:var(--text);overflow:hidden;-webkit-font-smoothing:antialiased}
  #app{display:flex;height:100vh;width:100vw}
  aside{background:var(--panel);display:flex;flex-direction:column;overflow-y:auto;flex-shrink:0;z-index:5}
  #panel-izq{width:322px;border-right:1px solid var(--line)}
  #panel-der{width:360px;border-left:1px solid var(--line)}
  aside::-webkit-scrollbar{width:8px}
  aside::-webkit-scrollbar-thumb{background:#cfd6e0;border-radius:8px}
  .brand{display:flex;gap:11px;align-items:center;padding:16px 18px;border-bottom:1px solid var(--line)}
  .logo{width:38px;height:38px;border-radius:9px;flex-shrink:0;
        background:linear-gradient(135deg,#2563eb,#1e40af);color:#fff;font-weight:800;font-size:14px;
        display:flex;align-items:center;justify-content:center;box-shadow:0 3px 8px rgba(37,99,235,.3)}
  .brand h1{font-size:14.5px;margin:0;font-weight:700;letter-spacing:-.2px}
  .brand p{font-size:11.5px;margin:2px 0 0;color:var(--muted)}
  .bloque{padding:16px 18px;border-bottom:1px solid var(--line-soft)}
  .bloque h2{font-size:11px;text-transform:uppercase;letter-spacing:.09em;color:var(--muted);
             margin:0 0 11px;font-weight:700}
  .hint{font-size:11px;color:#64748b;line-height:1.45;margin:0 0 10px;
        background:#f0f5ff;border-left:3px solid #93c5fd;padding:7px 9px;border-radius:5px}
  .hint b{color:#1e40af}
  .selector{display:grid;grid-template-columns:1fr 1fr;gap:7px}
  .elem-btn{position:relative;display:flex;flex-direction:column;gap:6px;align-items:flex-start;
            padding:9px 9px 8px;background:#fbfcfe;border:1.5px solid var(--line);border-radius:9px;
            cursor:pointer;font-family:inherit;color:var(--text);transition:all .13s ease;text-align:left}
  .elem-btn:hover{border-color:#b9c6d8;background:#f4f7fb;transform:translateY(-1px)}
  .elem-btn.activo{border-color:var(--accent);background:#eff5ff;box-shadow:0 0 0 3px rgba(37,99,235,.11)}
  .elem-btn .sw{width:100%;height:7px;border-radius:4px;border:1px solid rgba(0,0,0,.14)}
  .elem-btn .nm{font-size:11px;font-weight:600;line-height:1.25}
  .elem-btn .kbd{position:absolute;top:6px;right:6px;font-size:9px;font-weight:700;color:#94a3b8;
                 background:#eef2f7;border-radius:4px;padding:1px 5px;line-height:1.5}
  .elem-btn.activo .kbd{background:#dbeafe;color:var(--accent-dark)}
  .elem-btn.cubicado{pointer-events:none;cursor:not-allowed;background:#f0fdf4;
    border-color:#86efac;opacity:.72}
  .elem-btn.cubicado .nm{color:#166534}
  .elem-btn.cubicado .kbd{background:#bbf7d0;color:#166534}
  .elem-btn.cubicado::before{content:"✓";position:absolute;top:6px;left:6px;
    width:16px;height:16px;border-radius:50%;background:#16a34a;color:#fff;
    font-size:11px;font-weight:900;line-height:16px;text-align:center;
    box-shadow:0 1px 3px rgba(22,163,74,.4)}
  .elem-btn.cubicado.warn::before{background:#ea580c}
  .elem-btn.cubicado.bad::before{background:#dc2626}
  .campo{display:block;margin-bottom:10px}
  .campo span{display:block;font-size:11.5px;font-weight:600;color:#475569;margin-bottom:4px}
  .campo input{width:100%;padding:8px 10px;font-size:13.5px;
               font-family:ui-monospace,SFMono-Regular,Menlo,monospace;
               border:1.5px solid var(--line);border-radius:8px;background:#fbfcfe;color:var(--text);
               transition:border-color .13s, box-shadow .13s;outline:none}
  .campo input:focus{border-color:var(--accent);background:#fff;box-shadow:0 0 0 3px rgba(37,99,235,.11)}
  .campo input.error{border-color:var(--bad);background:#fef2f2}
  .campo input.ok{border-color:var(--ok);background:#f0fdf4}
  .campo input::-webkit-outer-spin-button,.campo input::-webkit-inner-spin-button{-webkit-appearance:none;margin:0}
  .campo input[type=number]{-moz-appearance:textfield}
  .grid3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:7px}
  .grid3 .campo{margin-bottom:0}
  .grid3 .campo input{padding:8px 6px;font-size:12.5px;text-align:center}
  .grid3 .campo span{font-size:10px;text-align:center}
  .mini-btn{margin-top:8px;width:100%;padding:6px;font-size:11px;font-weight:600;
            background:#f4f6fa;border:1px dashed var(--line);color:#64748b;
            border-radius:7px;cursor:pointer;font-family:inherit;transition:all .13s}
  .mini-btn:hover{background:#eff5ff;border-color:#93c5fd;color:var(--accent-dark)}
  .formula{font-size:11px;color:var(--muted);margin:2px 0 12px;
           font-family:ui-monospace,SFMono-Regular,Menlo,monospace;
           background:#f6f8fb;border:1px dashed var(--line);padding:6px 9px;border-radius:7px;text-align:center}
  .btn-primario,.btn-secundario{width:100%;padding:11px;border-radius:9px;cursor:pointer;
       font-family:inherit;font-size:13px;font-weight:700;transition:all .13s ease;border:none}
  .btn-primario{background:linear-gradient(135deg,#3b82f6,#2563eb);color:#fff;
       box-shadow:0 3px 10px rgba(37,99,235,.32);margin-bottom:8px}
  .btn-primario:hover{background:linear-gradient(135deg,#2563eb,#1d4ed8);transform:translateY(-1px)}
  .btn-primario:disabled{background:#cbd5e1;box-shadow:none;cursor:not-allowed;transform:none}
  .btn-secundario{background:#f4f6fa;color:#475569;border:1.5px solid var(--line)}
  .btn-secundario:hover{background:#e9eef6;border-color:#c3cddb}
  .btn-secundario.terciario{margin-top:6px;font-size:12px;padding:9px}
  #panel-der > h2{font-size:11px;text-transform:uppercase;letter-spacing:.09em;color:var(--muted);
       margin:0;padding:16px 18px 12px;font-weight:700;border-bottom:1px solid var(--line-soft)}
  .badge{margin:16px 18px 0;padding:13px;border-radius:10px;text-align:center;font-weight:800;
       font-size:14px;letter-spacing:.3px;border:1.5px solid var(--line);background:#f6f8fb;
       color:var(--muted);transition:all .2s}
  .badge.ok{background:#ecfdf5;color:#15803d;border-color:#a7f3d0}
  .badge.warn{background:#fff7ed;color:#c2410c;border-color:#fed7aa}
  .badge.bad{background:#fef2f2;color:#b91c1c;border-color:#fecaca}
  .filas{padding:14px 18px 4px}
  .fila{display:flex;justify-content:space-between;align-items:baseline;padding:8px 0;
       border-bottom:1px dashed var(--line-soft);font-size:12.5px}
  .fila:last-child{border-bottom:none}
  .fila .k{color:var(--muted)}
  .fila .v{font-family:ui-monospace,SFMono-Regular,Menlo,monospace;font-weight:700;font-size:13px}
  .v.ok{color:var(--ok)}.v.warn{color:var(--warn)}.v.bad{color:var(--bad)}
  .barra-wrap{padding:10px 18px 14px}
  .barra-label{display:flex;justify-content:space-between;align-items:baseline;font-size:11px;
       font-weight:700;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:7px}
  .barra-label b{font-family:ui-monospace,monospace;font-size:14px;color:var(--text);letter-spacing:0}
  .barra{position:relative;height:16px;border-radius:8px;background:#eef1f6;
       border:1.5px solid var(--line);overflow:hidden}
  #barra-fill{position:absolute;inset:0 auto 0 0;width:0%;
       background:linear-gradient(90deg,#60a5fa,#2563eb);border-radius:6px;transition:width .08s linear}
  #barra-fill.ok{background:linear-gradient(90deg,#4ade80,#16a34a)}
  #barra-fill.warn{background:linear-gradient(90deg,#fbbf24,#ea580c)}
  #barra-fill.bad{background:linear-gradient(90deg,#f87171,#dc2626)}
  #barra-target{position:absolute;top:-3px;bottom:-3px;left:66.66%;width:2px;
       background:#0f172a;opacity:.55}
  #barra-target::after{content:"100%";position:absolute;top:-15px;left:50%;transform:translateX(-50%);
       font-size:8.5px;font-weight:800;color:#0f172a;opacity:.7;white-space:nowrap}
  .feedback{margin:0 18px 18px;padding:12px 13px;border-radius:9px;font-size:12px;line-height:1.55;
       background:#f6f8fb;border:1px solid var(--line-soft);color:#475569;min-height:64px}
  .feedback.ok{background:#f0fdf4;border-color:#bbf7d0;color:#166534}
  .feedback.warn{background:#fff7ed;border-color:#fed7aa;color:#9a3412}
  .feedback.bad{background:#fef2f2;border-color:#fecaca;color:#991b1b}
  .feedback b{font-weight:800}
  #cost-panel{margin:0 18px 18px;padding:13px 15px 14px;
    background:linear-gradient(135deg,#ffffff 0%,#f6faff 100%);
    border:1px solid #dbe6f5;border-radius:12px;box-shadow:0 4px 14px rgba(15,23,42,.06)}
  #cost-panel .cost-title{display:flex;align-items:center;justify-content:space-between;gap:6px;
    font-size:11px;font-weight:800;letter-spacing:.06em;text-transform:uppercase;
    color:#1e40af;margin-bottom:9px}
  #cost-panel .cost-title .prog{font-family:ui-monospace,monospace;font-size:11px;
    background:#dbeafe;color:#1d4ed8;border-radius:20px;padding:2px 9px;letter-spacing:0}
  #cost-panel .cost-row{display:flex;justify-content:space-between;align-items:baseline;
    padding:5px 0;font-size:11.5px;color:#475569;border-bottom:1px dashed #e4ebf5}
  #cost-panel .cost-row:last-child{border-bottom:none}
  #cost-panel .cost-row .lbl{font-weight:600}
  #cost-panel .cost-row .num{font-family:ui-monospace,SFMono-Regular,Menlo,monospace;
    font-weight:700;font-size:13px;color:#0f172a;letter-spacing:-.02em}
  #cost-panel .cost-row .num small{font-size:9.5px;font-weight:700;color:#94a3b8;margin-left:3px;letter-spacing:0}
  #cost-panel .cost-row.total{margin-top:6px;padding-top:9px;
    border-top:1.5px solid #c7d7ee;border-bottom:none}
  #cost-panel .cost-row.total .lbl{color:#0f172a;font-weight:800;font-size:12px}
  #cost-panel .cost-row.total .num{font-size:16px;color:#047857}
  #cost-panel .cost-row.loss{margin-top:4px;background:#fef2f2;border-radius:6px;
    padding:6px 9px;border:none}
  #cost-panel .cost-row.loss .lbl{color:#b91c1c;font-weight:800}
  #cost-panel .cost-row.loss .num{color:#b91c1c;font-size:13px}
  #cost-panel .cost-row.saving{margin-top:4px;background:#ecfdf5;border-radius:6px;
    padding:6px 9px;border:none}
  #cost-panel .cost-row.saving .lbl{color:#15803d;font-weight:800}
  #cost-panel .cost-row.saving .num{color:#15803d;font-size:13px}
  #cost-panel .cost-row.balance{margin-top:6px;padding:9px 10px;border-radius:8px;
    border:none;background:#f1f5f9}
  #cost-panel .cost-row.balance .lbl{font-weight:800;font-size:11.5px}
  #cost-panel .cost-row.balance .num{font-size:15px}
  #viewport{flex:1;position:relative;min-width:0;background:#dbe2ea}
  #canvas{display:block;width:100%;height:100%;cursor:grab}
  #canvas:active{cursor:grabbing}
  .hud{position:absolute;pointer-events:none;font-size:11px;color:#64748b}
  #hud-top{top:14px;left:16px;background:rgba(255,255,255,.9);backdrop-filter:blur(6px);
       border:1px solid var(--line);border-radius:9px;padding:8px 13px;box-shadow:var(--shadow);
       display:flex;gap:14px;align-items:center}
  #hud-top .hud-main b{color:var(--text);font-size:12px;display:block}
  #hud-top .hud-main .sub{font-size:10.5px;margin-top:2px}
  #hud-top .hud-score{
    display:flex;gap:10px;padding-left:14px;border-left:1px solid var(--line);
    align-items:center;
  }
  #hud-top .score-item{text-align:center}
  #hud-top .score-item .v{
    font-family:ui-monospace,monospace;font-weight:800;font-size:15px;
    color:var(--accent-dark);letter-spacing:-.02em;line-height:1;
  }
  #hud-top .score-item .k{
    font-size:8.5px;text-transform:uppercase;letter-spacing:.08em;
    color:var(--muted);font-weight:700;margin-top:3px;
  }
  #hud-top .score-item.gold .v{color:#b45309}
  #hud-top .score-item.fire .v{color:#dc2626}
  #hud-bottom{bottom:14px;left:16px;right:16px;display:flex;justify-content:space-between;
       align-items:flex-end;gap:12px}
  .hud-card{background:rgba(255,255,255,.9);backdrop-filter:blur(6px);border:1px solid var(--line);
       border-radius:9px;padding:8px 13px;box-shadow:var(--shadow)}
  .hud-legend{display:flex;flex-wrap:wrap;gap:9px;max-width:520px}
  .hud-legend i{display:inline-block;width:9px;height:9px;border-radius:3px;margin-right:4px;
       vertical-align:-1px;border:1px solid rgba(0,0,0,.14)}
  .hud-btn{pointer-events:auto;cursor:pointer;font-family:inherit;font-size:11px;
       background:rgba(255,255,255,.95);border:1px solid var(--line);border-radius:7px;
       padding:6px 10px;color:#1f2937;font-weight:600;margin-left:6px}
  .hud-btn:hover{background:#eff5ff;border-color:#93c5fd}
  .hud-btn.sound{font-size:13px;padding:6px 9px}

  /* ============ PANEL DE LOGRO FLOTANTE ============ */
  #achievement{
    position:fixed; inset:0; z-index:9998;
    display:none; align-items:center; justify-content:center;
    pointer-events:none;
  }
  #achievement.show{display:flex}
  #achievement .backdrop{
    position:absolute; inset:0;
    background:radial-gradient(circle at center, rgba(15,23,42,.35) 0%, rgba(15,23,42,.55) 70%);
    pointer-events:auto;
    animation: fadeIn .3s ease;
  }
  #achievement .card{
    position:relative;
    pointer-events:auto;
    width:340px; padding:26px 24px 22px;
    background:#ffffff;
    border-radius:20px;
    box-shadow:0 24px 60px rgba(15,23,42,.4), 0 2px 6px rgba(15,23,42,.15);
    text-align:center;
    animation: popIn .55s cubic-bezier(.18,1.4,.4,1);
    overflow:hidden;
  }
  #achievement .card::before{
    content:"";
    position:absolute; inset:-2px;
    background:conic-gradient(from 0deg, #2563eb, #38bdf8, #a78bfa, #2563eb);
    border-radius:22px;
    z-index:-1;
    animation: spin 4s linear infinite;
    opacity:.85;
  }
  #achievement.exacto .card::before{background:conic-gradient(from 0deg, #22c55e, #4ade80, #fbbf24, #22c55e)}
  #achievement.warn .card::before{background:conic-gradient(from 0deg, #f59e0b, #fb923c, #fbbf24, #f59e0b)}
  #achievement.bad .card::before{background:conic-gradient(from 0deg, #dc2626, #f87171, #fbbf24, #dc2626)}
  #achievement .ring{
    width:96px; height:96px; margin:0 auto 6px;
    border-radius:50%;
    display:flex; align-items:center; justify-content:center;
    font-size:54px; line-height:1;
    position:relative;
    background:radial-gradient(circle, #ffffff 60%, #f1f5f9 100%);
    box-shadow: inset 0 2px 8px rgba(0,0,0,.08), 0 6px 20px rgba(37,99,235,.18);
    animation: bounceIn .7s cubic-bezier(.18,1.4,.4,1) .1s both;
  }
  #achievement.exacto .ring{box-shadow: inset 0 2px 8px rgba(0,0,0,.08), 0 8px 26px rgba(22,163,74,.35)}
  #achievement.warn .ring{box-shadow: inset 0 2px 8px rgba(0,0,0,.08), 0 8px 26px rgba(234,88,12,.3)}
  #achievement.bad .ring{box-shadow: inset 0 2px 8px rgba(0,0,0,.08), 0 8px 26px rgba(220,38,38,.3)}
  #achievement .ring::after{
    content:"";
    position:absolute; inset:-8px;
    border-radius:50%;
    border:3px solid transparent;
    border-top-color:rgba(37,99,235,.35);
    border-right-color:rgba(37,99,235,.15);
    animation: spin 3s linear infinite;
  }
  #achievement.exacto .ring::after{border-top-color:rgba(22,163,74,.4); border-right-color:rgba(245,158,11,.3)}
  #achievement.warn .ring::after{border-top-color:rgba(234,88,12,.4)}
  #achievement.bad .ring::after{border-top-color:rgba(220,38,38,.4)}

  #achievement .title{
    font-size:19px; font-weight:800; margin:10px 0 4px;
    letter-spacing:-.3px; color:#0f172a;
    animation: slideUp .45s ease .15s both;
  }
  #achievement.exacto .title{color:#15803d}
  #achievement.warn .title{color:#c2410c}
  #achievement.bad .title{color:#b91c1c}
  #achievement .subtitle{
    font-size:12px; color:#64748b; margin:0 0 12px;
    animation: slideUp .45s ease .22s both;
  }
  #achievement .stars{
    display:flex; justify-content:center; gap:6px; margin:6px 0 14px;
  }
  #achievement .stars span{
    font-size:26px; line-height:1;
    opacity:0; transform:scale(.4) rotate(-45deg);
    animation: starPop .55s cubic-bezier(.18,1.4,.4,1) forwards;
  }
  #achievement .stars span:nth-child(1){animation-delay:.35s}
  #achievement .stars span:nth-child(2){animation-delay:.48s}
  #achievement .stars span:nth-child(3){animation-delay:.61s}
  #achievement .stars span.on{color:#f59e0b; text-shadow:0 3px 10px rgba(245,158,11,.5)}
  #achievement .stars span.off{color:#cbd5e1}

  #achievement .points{
    display:inline-flex; align-items:center; gap:8px;
    padding:8px 16px; border-radius:30px;
    background:linear-gradient(135deg,#dbeafe,#eff6ff);
    font-weight:800; font-size:15px; color:#1d4ed8;
    border:1px solid #bfdbfe;
    margin:0 auto 12px;
    animation: slideUp .45s ease .5s both;
  }
  #achievement.exacto .points{background:linear-gradient(135deg,#dcfce7,#f0fdf4);color:#15803d;border-color:#bbf7d0}
  #achievement.warn .points{background:linear-gradient(135deg,#ffedd5,#fff7ed);color:#c2410c;border-color:#fed7aa}
  #achievement.bad .points{background:linear-gradient(135deg,#fee2e2,#fef2f2);color:#b91c1c;border-color:#fecaca}

  #achievement .msg{
    font-size:12px; line-height:1.55; color:#475569;
    padding:10px 12px; background:#f8fafc; border-radius:9px;
    border:1px dashed #e2e8f0; text-align:left;
    animation: slideUp .45s ease .55s both;
  }
  #achievement .msg b{color:#0f172a}
  #achievement .btn-cont{
    margin-top:14px; width:100%; padding:11px;
    border-radius:10px; border:none; cursor:pointer;
    font-family:inherit; font-size:13px; font-weight:800; letter-spacing:.02em;
    background:linear-gradient(135deg,#3b82f6,#2563eb); color:#fff;
    box-shadow:0 4px 12px rgba(37,99,235,.35);
    transition: transform .15s ease;
    animation: slideUp .45s ease .7s both;
  }
  #achievement.exacto .btn-cont{background:linear-gradient(135deg,#22c55e,#16a34a);box-shadow:0 4px 12px rgba(22,163,74,.35)}
  #achievement.warn .btn-cont{background:linear-gradient(135deg,#f59e0b,#ea580c);box-shadow:0 4px 12px rgba(234,88,12,.35)}
  #achievement.bad .btn-cont{background:linear-gradient(135deg,#f87171,#dc2626);box-shadow:0 4px 12px rgba(220,38,38,.35)}
  #achievement .btn-cont:hover{transform:translateY(-1px)}

  #achievement .confetti{
    position:absolute; top:0; left:0; width:100%; height:100%;
    pointer-events:none; overflow:hidden; border-radius:20px;
  }
  #achievement .confetti i{
    position:absolute; top:-10px;
    width:8px; height:14px; border-radius:2px;
    opacity:0;
    animation: fall 3s linear forwards;
  }

  @keyframes fadeIn{from{opacity:0}to{opacity:1}}
  @keyframes popIn{
    0%{opacity:0; transform:scale(.7) translateY(20px)}
    100%{opacity:1; transform:scale(1) translateY(0)}
  }
  @keyframes bounceIn{
    0%{opacity:0; transform:scale(.3)}
    60%{opacity:1; transform:scale(1.15)}
    100%{transform:scale(1)}
  }
  @keyframes slideUp{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
  @keyframes spin{to{transform:rotate(360deg)}}
  @keyframes starPop{
    0%{opacity:0;transform:scale(.4) rotate(-45deg)}
    70%{opacity:1;transform:scale(1.2) rotate(8deg)}
    100%{opacity:1;transform:scale(1) rotate(0)}
  }
  @keyframes fall{
    0%{opacity:0; transform:translateY(0) rotate(0)}
    10%{opacity:1}
    100%{opacity:0; transform:translateY(450px) rotate(720deg)}
  }

  /* ============ INFORME FINAL DE CUBICACIONES ============ */
  #informe-final{
    position:fixed; inset:0; z-index:9997;
    display:none; align-items:flex-start; justify-content:center;
    background:rgba(15,23,42,.88);
    padding:24px; overflow-y:auto;
  }
  #informe-final.show{display:flex}
  #informe-final .informe{
    background:#fff; max-width:1080px; width:100%;
    border-radius:14px; box-shadow:0 24px 60px rgba(0,0,0,.5);
    overflow:hidden; margin:auto;
  }
  .inf-header{
    padding:24px 36px 20px; border-bottom:3px solid #1e40af;
    background:linear-gradient(135deg,#f8fafc 0%,#eff6ff 100%);
    display:flex; justify-content:space-between; align-items:flex-start; gap:20px;
  }
  .inf-eyebrow{font-size:10px;font-weight:800;letter-spacing:.18em;color:#1e40af;margin-bottom:6px}
  .inf-title{font-size:22px;font-weight:800;color:#0f172a;margin:0 0 4px;letter-spacing:-.3px}
  .inf-sub{font-size:12.5px;color:#64748b;margin:0}
  .inf-meta{text-align:right;font-size:11px;color:#475569;line-height:1.6;
    border-left:1px solid #cbd5e1;padding-left:16px;min-width:190px}
  .inf-meta .lbl{font-size:9.5px;text-transform:uppercase;letter-spacing:.08em;color:#94a3b8;font-weight:700}
  .inf-body{padding:24px 36px 20px}
  .inf-section{margin-bottom:26px}
  .inf-section h2{font-size:12px;text-transform:uppercase;letter-spacing:.1em;
    color:#1e40af;font-weight:800;margin:0 0 14px;padding-bottom:7px;border-bottom:1.5px solid #dbeafe}
  .inf-kpis{display:grid;grid-template-columns:repeat(3,1fr);gap:10px}
  .kpi{background:#f8fafc;border:1px solid #e2e8f0;border-radius:10px;padding:11px 14px}
  .kpi-lbl{font-size:10px;text-transform:uppercase;letter-spacing:.06em;
    color:#64748b;font-weight:700;margin-bottom:4px}
  .kpi-val{font-family:ui-monospace,monospace;font-size:16px;font-weight:800;
    color:#0f172a;letter-spacing:-.02em}
  .kpi-val small{font-size:11px;color:#94a3b8;font-weight:700}
  .inf-tabla{width:100%;border-collapse:collapse;font-size:11.5px;
    border:1px solid #e2e8f0;border-radius:8px;overflow:hidden}
  .inf-tabla th{background:#1e40af;color:#fff;font-weight:700;padding:9px 8px;
    text-align:center;font-size:10.5px;letter-spacing:.03em;
    border-right:1px solid rgba(255,255,255,.15)}
  .inf-tabla th:last-child{border-right:none}
  .inf-tabla td{padding:9px 8px;border-bottom:1px solid #e2e8f0;
    text-align:center;vertical-align:middle}
  .inf-tabla td:first-child{text-align:left}
  .inf-tabla tbody tr:nth-child(even){background:#f8fafc}
  .inf-tabla tbody tr:hover{background:#eff6ff}
  .inf-tabla .mono{font-family:ui-monospace,SFMono-Regular,Menlo,monospace;font-size:11px}
  .inf-tabla .dim-small{font-size:9.5px;color:#94a3b8;font-weight:600}
  .inf-tabla tfoot td{background:#eff6ff;border-top:2px solid #1e40af;
    border-bottom:none;padding:11px 8px}
  .inf-desempeno{display:grid;grid-template-columns:repeat(5,1fr);gap:10px;margin-bottom:12px}
  .desemp-item{background:linear-gradient(135deg,#f8fafc,#f1f5f9);
    border:1px solid #e2e8f0;border-radius:10px;padding:11px 8px;text-align:center}
  .desemp-lbl{font-size:9.5px;text-transform:uppercase;letter-spacing:.06em;
    color:#64748b;font-weight:700;margin-bottom:5px}
  .desemp-val{font-family:ui-monospace,monospace;font-size:18px;font-weight:800;letter-spacing:-.02em}
  .inf-desglose{display:flex;gap:8px;flex-wrap:wrap}
  .chip{font-size:11px;font-weight:700;padding:4px 11px;border-radius:20px}
  .chip.ok{background:#dcfce7;color:#15803d}
  .chip.warn{background:#ffedd5;color:#c2410c}
  .chip.bad{background:#fee2e2;color:#b91c1c}
  .inf-obs p{font-size:12px;line-height:1.6;color:#475569;margin:0 0 9px}
  .inf-obs p:last-child{margin-bottom:0}
  .inf-obs b{color:#0f172a}
  .inf-firma{margin-top:36px;text-align:center;display:flex;flex-direction:column;align-items:center}
  .firma-linea{width:280px;border-top:1.5px solid #94a3b8;margin-bottom:6px}
  .firma-lbl{font-size:10.5px;color:#64748b;text-transform:uppercase;
    letter-spacing:.1em;font-weight:700}
  .inf-acciones{display:flex;gap:10px;justify-content:flex-end;
    padding:16px 36px 22px;border-top:1px solid #e2e8f0;background:#f8fafc}
  .inf-btn{padding:10px 20px;border-radius:9px;cursor:pointer;font-family:inherit;
    font-size:12.5px;font-weight:700;border:1.5px solid transparent;transition:all .15s}
  .inf-btn.secundario{background:#fff;color:#334155;border-color:#cbd5e1}
  .inf-btn.secundario:hover{background:#f1f5f9;border-color:#94a3b8}
  .inf-btn.primario{background:linear-gradient(135deg,#3b82f6,#2563eb);
    color:#fff;box-shadow:0 3px 10px rgba(37,99,235,.3)}
  .inf-btn.primario:hover{transform:translateY(-1px)}

  #error-msg{display:none;position:fixed;inset:0;background:rgba(15,23,42,.85);z-index:9999;
       align-items:center;justify-content:center;padding:30px}
  #error-msg > div{background:#fff;border-radius:14px;max-width:520px;padding:26px 30px;
       box-shadow:0 20px 60px rgba(0,0,0,.4);font-size:14px;line-height:1.6;color:#1f2937}
  #error-msg h3{margin:0 0 10px;color:#dc2626;font-size:17px}
  #error-msg code{background:#f1f5f9;padding:2px 6px;border-radius:5px;
       font-family:ui-monospace,monospace;font-size:12.5px}
  @media (max-width:1180px){#panel-izq{width:280px}#panel-der{width:320px}}

  @media print{
    body > *:not(#informe-final){display:none !important}
    #informe-final{display:block !important;position:static;background:#fff;padding:0;inset:auto;overflow:visible}
    #informe-final .informe{max-width:none;width:100%;border-radius:0;box-shadow:none;margin:0}
    .inf-acciones{display:none !important}
    .inf-header{background:#fff}
    .inf-tabla th,.kpi,.desemp-item,.chip,.inf-tabla tfoot td{
      -webkit-print-color-adjust:exact;print-color-adjust:exact}
html,body{margin:0!important;padding:0!important;max-width:none!important}
#app{min-height:600px;min-width:900px}
@media (max-width:1180px){
  #panel-izq{width:260px}
  #panel-der{width:290px}
}
@media (max-width:960px){
  #app{min-width:100%;flex-direction:column;height:auto;min-height:100vh}
  #panel-izq,#panel-der{width:100%;border-left:none;border-right:none;border-bottom:1px solid var(--line)}
  #viewport{min-height:420px;flex:1 0 420px}
}
  }
</style>
</head>
<body>
<div id="app">

  <aside id="panel-izq">
    <div class="brand">
      <div class="logo">HC</div>
      <div><h1>Cubicación de Hormigón</h1><p>Simulador 3D · vista isométrica</p></div>
    </div>

    <section class="bloque">
      <h2>1 · Elemento estructural</h2>
      <div id="selector" class="selector"></div>
    </section>

    <section class="bloque">
      <h2>2 · Lee las cotas del modelo</h2>
      <p class="hint">
        📐 Las cotas del modelo 3D pueden estar en <b>m</b>, <b>cm</b> o <b>mm</b>.
        Conviértelas a <b>metros</b> y escríbelas. El sistema verificará tu lectura.
      </p>
      <div class="grid3">
        <label class="campo"><span id="lbl-largo">Largo (m)</span>
          <input id="in-largo" data-dim="largo" type="number" step="0.001" min="0"
                 placeholder="0.00" value=""></label>
        <label class="campo"><span id="lbl-ancho">Ancho (m)</span>
          <input id="in-ancho" data-dim="ancho" type="number" step="0.001" min="0"
                 placeholder="0.00" value=""></label>
        <label class="campo"><span id="lbl-alto">Alto (m)</span>
          <input id="in-alto" data-dim="alto" type="number" step="0.001" min="0"
                 placeholder="0.00" value=""></label>
      </div>
      <p id="aviso-dim" class="hint" style="display:none;margin-top:8px;background:#fef2f2;border-left-color:#dc2626;color:#991b1b"></p>
      <p id="nota-detalle" class="hint" style="display:none;margin-top:8px;background:#fef3c7;border-left-color:#f59e0b;color:#78350f">
        🏗️ <b>Detalle de fundación:</b> el <b>emplantillado</b> y el <b>cimiento</b>
        forman una base continua bajo el muro y el pilar.
      </p>
      <p id="nota-muro" class="hint" style="display:none;margin-top:8px;background:#ecfeff;border-left-color:#06b6d4;color:#155e75">
        🪟 <b>Muro con vano:</b> <span id="nota-vano-texto"></span>
      </p>
      <p id="nota-pilar" class="hint" style="display:none;margin-top:8px;background:#f0fdf4;border-left-color:#16a34a;color:#14532d">
        🏛️ <b>Pilar:</b> este pilar <b>nace del cimiento</b>, no flota.
      </p>
      <button id="btn-vaciar" class="mini-btn">✎ Vaciar casillas</button>
    </section>

    <section class="bloque">
      <h2>3 · Volumen total de hormigón</h2>
      <p style="font-size:11.5px;color:#64748b;margin:0 0 8px;line-height:1.45">
        Calcula el volumen total con las dimensiones en metros y escríbelo en m³.
      </p>
      <label class="campo"><span>Volumen total (m³)</span>
        <input id="in-vol" type="number" step="0.001" min="0" placeholder="0.000"></label>
      <p class="formula" id="formula-viva">V = ? × ? × ? = ? m³</p>
      <button id="btn-verter" class="btn-primario">▶ Calcular y verter</button>
      <button id="btn-centrar" class="btn-secundario terciario">⌖ Centrar vista en el elemento</button>
      <button id="btn-reset" class="btn-secundario terciario">↺ Nueva partida (reinicia proyecto)</button>
      <button id="btn-informe" class="btn-secundario terciario"
        style="display:none;background:linear-gradient(135deg,#eef2ff,#e0e7ff);
               border-color:#c7d2fe;color:#3730a3;font-weight:800">
        📄 Ver informe de cubicaciones
      </button>
    </section>
  </aside>

  <main id="viewport">
    <canvas id="canvas"></canvas>
    <div id="hud-top" class="hud">
      <div class="hud-main">
        <b id="hud-elem">Muro de hormigón armado</b>
        <div class="sub" id="hud-dims">Lee las cotas del modelo →</div>
      </div>
      <div class="hud-score">
        <div class="score-item">
          <div class="v" id="hud-pts">0</div>
          <div class="k">Puntos</div>
        </div>
        <div class="score-item fire">
          <div class="v" id="hud-racha">0</div>
          <div class="k">Racha</div>
        </div>
        <div class="score-item gold">
          <div class="v" id="hud-estrellas">0</div>
          <div class="k">Estrellas</div>
        </div>
      </div>
    </div>
    <div id="hud-bottom" class="hud">
      <div class="hud-card hud-legend" id="leyenda"></div>
      <div class="hud-card">
        🖱️ Arrastrar: rotar · Rueda: zoom · ⌨️ Teclas 1–7: seleccionar · C: centrar
        <button class="hud-btn" id="btn-centrar-hud">⌖ Centrar</button>
        <button class="hud-btn sound" id="btn-sound" title="Activar/Silenciar">🔊</button>
      </div>
    </div>
  </main>

  <aside id="panel-der">
    <h2>Resultado de la comparación</h2>
    <div id="badge" class="badge">Sin calcular</div>
    <div class="filas">
      <div class="fila"><span class="k">Volumen real del elemento</span><span class="v" id="r-real">—</span></div>
      <div class="fila"><span class="k">Tu estimación</span><span class="v" id="r-est">—</span></div>
      <div class="fila"><span class="k">Diferencia</span><span class="v" id="r-dif">—</span></div>
      <div class="fila"><span class="k">Error relativo</span><span class="v" id="r-pct">—</span></div>
    </div>
    <div class="barra-wrap">
      <div class="barra-label"><span>Nivel de llenado</span><b id="pct-llenado">0 %</b></div>
      <div class="barra"><div id="barra-fill"></div><div id="barra-target"></div></div>
    </div>
    <div id="feedback" class="feedback">
      Lee las <b>cotas del modelo</b> (pueden estar en m, cm o mm),
      conviértelas a metros, calcula el volumen y escríbelo.
    </div>

    <div id="cost-panel">
      <div class="cost-title">
        <span>💰 Costo del hormigón · G25</span>
        <span class="prog" id="cost-prog">0 / 7</span>
      </div>
      <div class="cost-row">
        <span class="lbl">Precio unitario</span>
        <span class="num">$115.000 <small>CLP/m³</small></span>
      </div>
      <div class="cost-row">
        <span class="lbl">Volumen real cubicado</span>
        <span class="num" id="cost-vol-real">0.000 <small>m³</small></span>
      </div>
      <div class="cost-row">
        <span class="lbl">Tu volumen cubicado</span>
        <span class="num" id="cost-vol-est">0.000 <small>m³</small></span>
      </div>
      <div class="cost-row total">
        <span class="lbl">💵 Costo real acumulado</span>
        <span class="num" id="cost-real">$0</span>
      </div>
      <div class="cost-row" id="cost-diff-row">
        <span class="lbl">Tu costo acumulado</span>
        <span class="num" id="cost-est">$0</span>
      </div>
      <div class="cost-row loss" id="cost-loss-row" style="display:none">
        <span class="lbl">🔻 Sobrecosto por exceso</span>
        <span class="num" id="cost-loss">$0</span>
      </div>
      <div class="cost-row saving" id="cost-save-row" style="display:none">
        <span class="lbl">💚 Ahorro por defecto</span>
        <span class="num" id="cost-save">$0</span>
      </div>
      <div class="cost-row balance" id="cost-balance-row" style="display:none">
        <span class="lbl">📊 Balance final proyecto</span>
        <span class="num" id="cost-balance">$0</span>
      </div>
    </div>
  </aside>
</div>

<!-- ============ PANEL DE LOGRO FLOTANTE ============ -->
<div id="achievement">
  <div class="backdrop" id="ach-backdrop"></div>
  <div class="card">
    <div class="confetti" id="ach-confetti"></div>
    <div class="ring" id="ach-icon">🏆</div>
    <div class="title" id="ach-title">¡Excelente!</div>
    <div class="subtitle" id="ach-sub">Cubicación exacta</div>
    <div class="stars" id="ach-stars">
      <span>★</span><span>★</span><span>★</span>
    </div>
    <div class="points" id="ach-points">+100 pts</div>
    <div class="msg" id="ach-msg"></div>
    <button class="btn-cont" id="ach-btn">Continuar ▶</button>
  </div>
</div>

<!-- ============ INFORME FINAL ============ -->
<div id="informe-final">
  <div class="informe" id="informe-contenido"></div>
</div>

<div id="error-msg"><div>
  <h3>No se pudo cargar Three.js</h3>
  <p>Este simulador necesita conexión a internet para cargar la librería 3D, y debe abrirse
     desde un <b>servidor local</b> (no con <code>file://</code>).</p>
  <p>Ejecuta en la carpeta del archivo:</p>
  <p><code>python -m http.server 8000</code></p>
  <p>Y abre <code>http://localhost:8000/</code></p>
</div></div>

<script>
window.addEventListener('error', function(e){
  if (e && e.message && /import|module|three/i.test(e.message)) {
    document.getElementById('error-msg').style.display = 'flex';
  }
}, true);
setTimeout(() => {
  if (!window.__threeOK) document.getElementById('error-msg').style.display = 'flex';
}, 4000);
</script>

<script type="importmap">
{ "imports": { "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js" } }
</script>

<script type="module">
import * as THREE from 'three';
window.__threeOK = true;

/* =========================================================
   0. SISTEMA DE SONIDO (Web Audio API, sin archivos)
   ========================================================= */
let audioCtx = null;
let sonidoActivo = true;

function initAudio(){
  if (!audioCtx){
    try {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    } catch(e){ audioCtx = null; }
  }
  if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
}

function tone(freq, startOffset, dur, type='sine', vol=0.15){
  if (!audioCtx || !sonidoActivo) return;
  const t0 = audioCtx.currentTime + startOffset;
  const osc = audioCtx.createOscillator();
  const g   = audioCtx.createGain();
  osc.type = type;
  osc.frequency.setValueAtTime(freq, t0);
  g.gain.setValueAtTime(0, t0);
  g.gain.linearRampToValueAtTime(vol, t0 + 0.012);
  g.gain.exponentialRampToValueAtTime(0.0001, t0 + dur);
  osc.connect(g).connect(audioCtx.destination);
  osc.start(t0); osc.stop(t0 + dur + 0.05);
}

function sonidoExacto(){
  initAudio();
  tone(523.25, 0.00, 0.18, 'sine', 0.20);
  tone(659.25, 0.11, 0.18, 'sine', 0.20);
  tone(783.99, 0.22, 0.22, 'sine', 0.20);
  tone(1046.5, 0.36, 0.40, 'triangle', 0.22);
}
function sonidoFalta(){
  initAudio();
  tone(440, 0.00, 0.22, 'sine', 0.15);
  tone(369.99, 0.18, 0.30, 'sine', 0.15);
}
function sonidoDesborda(){
  initAudio();
  tone(233.08, 0.00, 0.22, 'sawtooth', 0.09);
  tone(174.61, 0.15, 0.35, 'sawtooth', 0.09);
  tone(116.54, 0.30, 0.45, 'square',   0.07);
}
function sonidoClick(){
  initAudio();
  tone(880, 0, 0.045, 'square', 0.05);
}
function sonidoError(){
  initAudio();
  tone(196, 0.00, 0.18, 'square', 0.10);
  tone(146.83, 0.14, 0.30, 'square', 0.10);
}
function sonidoVictoria(){
  initAudio();
  const notes = [523.25, 659.25, 783.99, 1046.5, 1318.51];
  notes.forEach((n, i) => tone(n, i * 0.10, 0.32, 'triangle', 0.20));
  tone(1046.5, 0.60, 0.7, 'sine',     0.18);
  tone(1318.51, 0.60, 0.7, 'sine',    0.15);
  tone(1567.98, 0.60, 0.9, 'triangle', 0.15);
}
function sonidoEstrella(indice){
  initAudio();
  tone(880 + indice * 220, 0, 0.15, 'triangle', 0.12);
}

/* =========================================================
   0b. PRECIO DEL HORMIGÓN (Chile)
   ========================================================= */
const PRECIO_HORMIGON_CLP = 115000;
function formatearCLP(v){
  const n = Math.round(v);
  const signo = n < 0 ? '-' : '';
  return signo + '$' + Math.abs(n).toLocaleString('es-CL');
}

/* =========================================================
   0c. ESTADO DE GAMIFICACIÓN
   ========================================================= */
const juego = {
  puntos: 0,
  aciertos: 0,
  intentos: 0,
  rachaActual: 0,
  mejorRacha: 0,
  estrellasTotales: 0,
  elementosCubicados: 0,
  proyectoCompletado: false
};

function actualizarHUDGamificacion(){
  document.getElementById('hud-pts').textContent      = juego.puntos;
  document.getElementById('hud-racha').textContent    = juego.rachaActual;
  document.getElementById('hud-estrellas').textContent = juego.estrellasTotales;
}

function reiniciarGamificacion(){
  juego.puntos = 0;
  juego.aciertos = 0;
  juego.intentos = 0;
  juego.rachaActual = 0;
  juego.mejorRacha = 0;
  juego.estrellasTotales = 0;
  juego.elementosCubicados = 0;
  juego.proyectoCompletado = false;
  actualizarHUDGamificacion();
}

/* =========================================================
   0d. PANEL DE LOGRO
   ========================================================= */
const ach = {
  el: document.getElementById('achievement'),
  icon: document.getElementById('ach-icon'),
  title: document.getElementById('ach-title'),
  sub: document.getElementById('ach-sub'),
  stars: document.getElementById('ach-stars'),
  points: document.getElementById('ach-points'),
  msg: document.getElementById('ach-msg'),
  btn: document.getElementById('ach-btn'),
  confetti: document.getElementById('ach-confetti')
};

function mostrarLogro(tipo, icono, titulo, subtitulo, estrellas, puntos, mensaje, opciones = {}){
  ach.el.classList.remove('exacto','warn','bad');
  ach.el.classList.add('show');
  if (tipo) ach.el.classList.add(tipo);

  ach.icon.textContent = icono;
  ach.title.textContent = titulo;
  ach.sub.textContent  = subtitulo;
  ach.points.textContent = (puntos >= 0 ? '+' : '') + puntos + ' pts';
  ach.msg.innerHTML = mensaje;

  ach.stars.innerHTML = '';
  for (let i = 0; i < 3; i++){
    const s = document.createElement('span');
    s.textContent = '★';
    s.className = i < estrellas ? 'on' : 'off';
    ach.stars.appendChild(s);
  }

  setTimeout(() => { if (estrellas >= 1) sonidoEstrella(0); }, 400);
  if (estrellas >= 2) setTimeout(() => sonidoEstrella(1), 560);
  if (estrellas >= 3) setTimeout(() => sonidoEstrella(2), 720);

  ach.confetti.innerHTML = '';
  if (opciones.confetti){
    const colores = ['#22c55e','#3b82f6','#f59e0b','#ec4899','#a855f7','#38bdf8'];
    for (let i = 0; i < 40; i++){
      const c = document.createElement('i');
      c.style.left = Math.random() * 100 + '%';
      c.style.background = colores[Math.floor(Math.random() * colores.length)];
      c.style.animationDelay = (Math.random() * 0.6) + 's';
      c.style.animationDuration = (2 + Math.random() * 1.5) + 's';
      c.style.transform = 'rotate(' + (Math.random() * 360) + 'deg)';
      ach.confetti.appendChild(c);
    }
  }

  ach.btn.textContent = opciones.textoBtn || 'Continuar ▶';
  ach.btn.dataset.abrirInforme = opciones.mostrarInforme ? '1' : '';
}

function ocultarLogro(){
  ach.el.classList.remove('show');
}

ach.btn.addEventListener('click', () => {
  sonidoClick();
  const abrirInf = ach.btn.dataset.abrirInforme === '1';
  ach.btn.dataset.abrirInforme = '';
  ocultarLogro();
  if (abrirInf) setTimeout(mostrarInforme, 260);
});
document.getElementById('ach-backdrop').addEventListener('click', () => ocultarLogro());
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && ach.el.classList.contains('show')) ocultarLogro();
  if (e.key === 'Enter' && ach.el.classList.contains('show')) { ocultarLogro(); e.stopPropagation(); }
});

/* =========================================================
   1. ELEMENTOS ESTRUCTURALES
   ========================================================= */
const ORDEN_UI = ['muro','losa','viga','pilar','emplantillado','cimiento','sobrecimiento'];

const ELEMENTOS = {
  muro: {
    nombre:'Muro de hormigón armado', corto:'Muro',
    color:'#d9dde2', colorHex:0xd9dde2,
    etiquetas:{ largo:'Largo', ancho:'Espesor', alto:'Altura' },
    dims:{ largo:4.00, ancho:0.18, alto:2.50 },
    vano:{ ancho:1.20, alto:1.05, alfeizar:1.10 }
  },
  losa: {
    nombre:'Losa de entrepiso', corto:'Losa',
    color:'#b9bfc6', colorHex:0xb9bfc6,
    etiquetas:{ largo:'Largo', ancho:'Ancho', alto:'Espesor' },
    dims:{ largo:4.00, ancho:3.00, alto:0.20 }
  },
  viga: {
    nombre:'Viga de coronamiento', corto:'Viga',
    color:'#8fa5bd', colorHex:0x8fa5bd,
    opacidadActiva: 0.82, opacidadInactiva: 0.48,
    etiquetas:{ largo:'Largo', ancho:'Base', alto:'Peralte' },
    dims:{ largo:4.00, ancho:0.18, alto:0.40 }
  },
  pilar: {
    nombre:'Pilar (columna)', corto:'Pilar',
    color:'#9aabb8', colorHex:0x9aabb8,
    etiquetas:{ largo:'Base X', ancho:'Base Z', alto:'Altura' },
    dims:{ largo:0.25, ancho:0.25, alto:3.20 }
  },
  emplantillado: {
    nombre:'Emplantillado (solado de limpieza)', corto:'Emplantillado',
    color:'#e3d5b8', colorHex:0xe3d5b8,
    etiquetas:{ largo:'Largo', ancho:'Ancho', alto:'Espesor' },
    dims:{ largo:4.50, ancho:0.70, alto:0.05 }
  },
  cimiento: {
    nombre:'Cimiento corrido', corto:'Cimiento',
    color:'#c8a882', colorHex:0xc8a882,
    etiquetas:{ largo:'Largo', ancho:'Ancho', alto:'Altura' },
    dims:{ largo:4.50, ancho:0.70, alto:0.60 }
  },
  sobrecimiento: {
    nombre:'Sobrecimiento', corto:'Sobrecimiento',
    color:'#a9bccb', colorHex:0xa9bccb,
    etiquetas:{ largo:'Largo', ancho:'Ancho', alto:'Altura' },
    dims:{ largo:4.00, ancho:0.20, alto:0.30 }
  }
};

const GRUPO_FUNDACION = ['emplantillado','cimiento','sobrecimiento','muro','pilar'];
const UNIDADES = ['m','cm','mm'];
const estadoProyecto = {};

/* =========================================================
   2. Utilidades
   ========================================================= */
function rand(min, max){
  return Math.max(0.02, Math.round((min + Math.random() * (max - min)) * 100) / 100);
}
function randUnit(){ return UNIDADES[Math.floor(Math.random() * UNIDADES.length)]; }
function formatearMedida(valorM, unidad){
  if (unidad === 'cm') return Math.round(valorM * 100) + ' cm';
  if (unidad === 'mm') return Math.round(valorM * 1000) + ' mm';
  return valorM.toFixed(2) + ' m';
}

function randomizarDimensiones(){
  const largo = rand(3.20, 4.80);
  const espMuro = rand(0.14, 0.22);
  const anchoBase = rand(0.55, 0.75);
  const anchoSobre = espMuro + rand(0.00, 0.05);

  ELEMENTOS.muro.dims.largo = largo;
  ELEMENTOS.muro.dims.ancho = espMuro;
  ELEMENTOS.muro.dims.alto  = rand(2.20, 2.80);

  ELEMENTOS.sobrecimiento.dims.largo = largo;
  ELEMENTOS.sobrecimiento.dims.ancho = anchoSobre;
  ELEMENTOS.sobrecimiento.dims.alto  = rand(0.28, 0.40);

  ELEMENTOS.cimiento.dims.ancho = anchoBase;
  ELEMENTOS.cimiento.dims.alto  = rand(0.50, 0.70);

  ELEMENTOS.emplantillado.dims.ancho = anchoBase;
  ELEMENTOS.emplantillado.dims.alto  = rand(0.04, 0.06);

  ELEMENTOS.viga.dims.largo = largo;
  ELEMENTOS.viga.dims.ancho = espMuro;
  ELEMENTOS.viga.dims.alto  = rand(0.35, 0.50);

  ELEMENTOS.losa.dims.largo = largo + rand(-0.20, 0.20);
  ELEMENTOS.losa.dims.ancho = rand(2.60, 3.40);
  ELEMENTOS.losa.dims.alto  = rand(0.18, 0.25);

  const baseP = rand(0.20, 0.30);
  ELEMENTOS.pilar.dims.largo = baseP;
  ELEMENTOS.pilar.dims.ancho = baseP;

  for (const key of ORDEN_UI){
    ELEMENTOS[key].unidades = { largo: randUnit(), ancho: randUnit(), alto: randUnit() };
  }
  ELEMENTOS.muro.unidadesVano = { ancho: randUnit(), alto: randUnit() };

  calcularPosiciones();
}

function calcularPosiciones(){
  const X0 = 0, Z0 = 0;
  const baseP  = ELEMENTOS.pilar.dims.largo;
  const pilarX = X0 + ELEMENTOS.muro.dims.largo / 2 + baseP / 2;

  const xLeft      = X0 - ELEMENTOS.muro.dims.largo / 2;
  const xRight     = pilarX + baseP / 2;
  const largoFund  = xRight - xLeft;
  const centroFund = (xLeft + xRight) / 2;

  ELEMENTOS.emplantillado.dims.largo = largoFund;
  ELEMENTOS.cimiento.dims.largo      = largoFund;
  ELEMENTOS.sobrecimiento.dims.largo = ELEMENTOS.muro.dims.largo;

  let y = 0;
  ELEMENTOS.emplantillado.pos = { x:centroFund, y, z:Z0 };  y += ELEMENTOS.emplantillado.dims.alto;
  ELEMENTOS.cimiento.pos      = { x:centroFund, y, z:Z0 };  y += ELEMENTOS.cimiento.dims.alto;
  const cimientoTop = y;

  ELEMENTOS.sobrecimiento.pos = { x:X0, y, z:Z0 };  y += ELEMENTOS.sobrecimiento.dims.alto;
  ELEMENTOS.muro.pos          = { x:X0, y, z:Z0 };  y += ELEMENTOS.muro.dims.alto;
  ELEMENTOS.viga.pos          = { x:X0, y, z:Z0 };  y += ELEMENTOS.viga.dims.alto;
  const vigaTop = y;

  ELEMENTOS.pilar.dims.alto = vigaTop - cimientoTop;
  ELEMENTOS.pilar.pos = { x:pilarX, y:cimientoTop, z:Z0 };

  const anchoViga = ELEMENTOS.viga.dims.ancho;
  const anchoLosa = ELEMENTOS.losa.dims.ancho;
  ELEMENTOS.losa.pos = {
    x: X0, y: vigaTop - ELEMENTOS.losa.dims.alto,
    z: Z0 + anchoViga / 2 + anchoLosa / 2
  };
}

/* =========================================================
   3. ESCENA Y CÁMARA
   ========================================================= */
const contenedor = document.getElementById('viewport');
const canvas     = document.getElementById('canvas');
const renderer = new THREE.WebGLRenderer({ canvas, antialias:true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.outputColorSpace = THREE.SRGBColorSpace;

const scene = new THREE.Scene();
scene.background = new THREE.Color(0xdbe2ea);

const cam = new THREE.OrthographicCamera(-1, 1, 1, -1, 0.1, 400);
let frustum = 11.0;
let theta   = Math.PI / 4;
let phi     = Math.atan(Math.SQRT2);
const RADIUS = 60;
const camCurrent = { target: new THREE.Vector3(0.4, 2.0, 0.6), frustum: 11.0 };
const camGoal    = { target: new THREE.Vector3(0.4, 2.0, 0.6), frustum: 11.0 };
let camLerp = 1;

function actualizarCamara(){
  const sp = Math.sin(phi), cp = Math.cos(phi);
  cam.position.set(
    camCurrent.target.x + RADIUS * sp * Math.cos(theta),
    camCurrent.target.y + RADIUS * cp,
    camCurrent.target.z + RADIUS * sp * Math.sin(theta)
  );
  cam.lookAt(camCurrent.target);
}

scene.add(new THREE.AmbientLight(0xffffff, 0.55));
scene.add(new THREE.HemisphereLight(0xffffff, 0x8fa0b3, 0.75));
const dir = new THREE.DirectionalLight(0xffffff, 1.15);
dir.position.set(10, 16, 8);
dir.castShadow = true;
dir.shadow.mapSize.set(2048, 2048);
dir.shadow.camera.left = -16; dir.shadow.camera.right = 16;
dir.shadow.camera.top = 16; dir.shadow.camera.bottom = -16;
dir.shadow.camera.near = 1; dir.shadow.camera.far = 50;
dir.shadow.bias = -0.0008;
scene.add(dir);
const fill = new THREE.DirectionalLight(0xdbe6f5, 0.45);
fill.position.set(-9, 7, -8);
scene.add(fill);

const suelo = new THREE.Mesh(
  new THREE.PlaneGeometry(80, 80),
  new THREE.MeshStandardMaterial({ color:0xffffff, roughness:1, metalness:0 })
);
suelo.rotation.x = -Math.PI / 2;
suelo.receiveShadow = true;
scene.add(suelo);

const grid = new THREE.GridHelper(80, 80, 0xc3ccd9, 0xe0e6ee);
grid.position.y = 0.005;
grid.material.transparent = true;
grid.material.opacity = 0.9;
scene.add(grid);

/* =========================================================
   4. SPRITES
   ========================================================= */
function roundRect(ctx, x, y, w, h, r){
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.arcTo(x + w, y, x + w, y + h, r);
  ctx.arcTo(x + w, y + h, x, y + h, r);
  ctx.arcTo(x, y + h, x, y, r);
  ctx.arcTo(x, y, x + w, y, r);
  ctx.closePath();
}
function crearSpriteEtiqueta(){
  const cv = document.createElement('canvas');
  cv.width = 512; cv.height = 128;
  const tex = new THREE.CanvasTexture(cv);
  tex.colorSpace = THREE.SRGBColorSpace;
  const mat = new THREE.SpriteMaterial({ map:tex, transparent:true, depthTest:false });
  const sp = new THREE.Sprite(mat);
  sp.scale.set(2.4, 0.60, 1);
  sp.renderOrder = 999;
  return sp;
}
function dibujarEtiqueta(sprite, l1, activo){
  const cv = sprite.material.map.image;
  const ctx = cv.getContext('2d');
  ctx.clearRect(0, 0, cv.width, cv.height);
  ctx.fillStyle = activo ? 'rgba(37,99,235,0.95)' : 'rgba(255,255,255,0.95)';
  roundRect(ctx, 5, 5, cv.width - 10, cv.height - 10, 20);
  ctx.fill();
  ctx.lineWidth = 3;
  ctx.strokeStyle = activo ? '#1d4ed8' : '#cbd5e1';
  ctx.stroke();
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillStyle = activo ? '#ffffff' : '#1f2937';
  ctx.font = 'bold 46px ui-sans-serif, system-ui, sans-serif';
  ctx.fillText(l1, cv.width / 2, cv.height / 2 + 4);
  sprite.material.map.needsUpdate = true;
}
function crearSpriteCota(colorFondo){
  const cv = document.createElement('canvas');
  cv.width = 256; cv.height = 64;
  const tex = new THREE.CanvasTexture(cv);
  tex.colorSpace = THREE.SRGBColorSpace;
  const mat = new THREE.SpriteMaterial({ map:tex, transparent:true, depthTest:false });
  const sp = new THREE.Sprite(mat);
  sp.scale.set(0.85, 0.22, 1);
  sp.renderOrder = 1000;
  sp.userData.colorFondo = colorFondo || 'rgba(185,28,28,0.96)';
  sp.userData.colorBorde = colorFondo ? '#1e40af' : '#7f1d1d';
  return sp;
}
function dibujarCota(sprite, texto){
  const cv = sprite.material.map.image;
  const ctx = cv.getContext('2d');
  ctx.clearRect(0, 0, cv.width, cv.height);
  ctx.fillStyle = sprite.userData.colorFondo;
  roundRect(ctx, 2, 2, cv.width - 4, cv.height - 4, 14);
  ctx.fill();
  ctx.strokeStyle = sprite.userData.colorBorde;
  ctx.lineWidth = 2;
  ctx.stroke();
  ctx.fillStyle = '#ffffff';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.font = 'bold 34px ui-monospace, monospace';
  ctx.fillText(texto, cv.width / 2, cv.height / 2 + 2);
  sprite.material.map.needsUpdate = true;
}

/* =========================================================
   5. OBJETOS 3D
   ========================================================= */
const objetos = {};
const grupoRaiz = new THREE.Group();
scene.add(grupoRaiz);
const GEO_UNIT   = new THREE.BoxGeometry(1, 1, 1);
const EDGES_UNIT = new THREE.EdgesGeometry(GEO_UNIT);
const GEO_FLECHA = new THREE.ConeGeometry(0.045, 0.14, 6);
const MAT_FLECHA = new THREE.MeshBasicMaterial({ color: 0xb91c1c });
const MAT_COTA_LINEA = new THREE.LineBasicMaterial({ color: 0xb91c1c, transparent:true, opacity:0.95 });
const MAT_FLECHA_VANO = new THREE.MeshBasicMaterial({ color: 0x1e40af });
const MAT_COTA_VANO_LINEA = new THREE.LineBasicMaterial({ color: 0x1e40af, transparent:true, opacity:0.95 });

function crearObjeto(key){
  const def = ELEMENTOS[key];
  const g = new THREE.Group();
  const molde = new THREE.Mesh(GEO_UNIT, new THREE.MeshStandardMaterial({
    color: def.colorHex, transparent: true, opacity: 0.96, depthWrite: true,
    roughness: 0.78, metalness: 0.02
  }));
  molde.castShadow = true;
  molde.receiveShadow = true;
  const aristas = new THREE.LineSegments(EDGES_UNIT, new THREE.LineBasicMaterial({
    color: 0x0f172a, transparent: true, opacity: 1
  }));
  aristas.renderOrder = 4;
  const concreto = new THREE.Mesh(GEO_UNIT, new THREE.MeshStandardMaterial({
    color: 0x8b9198, roughness: 0.96, metalness: 0.03
  }));
  concreto.castShadow = true;
  concreto.receiveShadow = true;
  concreto.visible = false;
  const etiqueta = crearSpriteEtiqueta();
  g.add(molde, aristas, concreto, etiqueta);
  grupoRaiz.add(g);
  objetos[key] = { grupo:g, molde, aristas, concreto, etiqueta, fillH:0 };
}
ORDEN_UI.forEach(crearObjeto);

const vanoMat = new THREE.MeshStandardMaterial({
  color: 0x1e3a52, roughness: 0.35, metalness: 0.15,
  emissive: 0x0a1828, emissiveIntensity: 0.4
});
const vanoMesh = new THREE.Mesh(GEO_UNIT, vanoMat);
vanoMesh.renderOrder = 10;
grupoRaiz.add(vanoMesh);
const marcoMat = new THREE.MeshStandardMaterial({ color: 0xf0f4f8, roughness: 0.6, metalness: 0 });
const marcoMesh = new THREE.Mesh(GEO_UNIT, marcoMat);
marcoMesh.renderOrder = 9;
grupoRaiz.add(marcoMesh);
const alfMat = new THREE.MeshStandardMaterial({ color: 0xdfe6ee, roughness: 0.7, metalness: 0 });
const alfMesh = new THREE.Mesh(GEO_UNIT, alfMat);
alfMesh.renderOrder = 9;
grupoRaiz.add(alfMesh);

function actualizarVano(){
  const m = ELEMENTOS.muro;
  const H = m.dims.alto, A = m.dims.ancho, L = m.dims.largo;
  const vAncho   = Math.min(L * 0.35, 1.60);
  const vAlto    = Math.min(H * 0.42, 1.30);
  const alfeizar = H * 0.42;
  m.vano.ancho = vAncho; m.vano.alto = vAlto; m.vano.alfeizar = alfeizar;
  const espesorVano = 0.020, espesorMarco = 0.05;
  const yCentro = m.pos.y + alfeizar + vAlto/2;
  const zFrente = m.pos.z + A/2 + 0.001;
  vanoMesh.scale.set(vAncho, vAlto, espesorVano);
  vanoMesh.position.set(m.pos.x, yCentro, zFrente + espesorVano/2);
  marcoMesh.scale.set(vAncho + espesorMarco*2, vAlto + espesorMarco*2, espesorVano * 0.6);
  marcoMesh.position.set(m.pos.x, yCentro, zFrente - espesorVano*0.3);
  alfMesh.scale.set(vAncho + 0.10, 0.05, A + 0.06);
  alfMesh.position.set(m.pos.x, yCentro - vAlto/2 - 0.025, m.pos.z + A/2 + 0.02);
  actualizarCotasVano();
  const notaTexto = document.getElementById('nota-vano-texto');
  if (notaTexto){
    const uv = m.unidadesVano || { ancho:'cm', alto:'cm' };
    notaTexto.innerHTML =
      `el vano mide <b>${formatearMedida(vAncho, uv.ancho)} × ${formatearMedida(vAlto, uv.alto)}</b> ` +
      `(alféizar a <b>${formatearMedida(alfeizar, 'm')}</b>).<br>` +
      `<b>Debes descontar el volumen del vano del muro completo.</b>`;
  }
}

/* =========================================================
   6. COTAS
   ========================================================= */
const cotas = {};
function crearCota(){
  const g = new THREE.Group();
  const lineaLargo = new THREE.Line(new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(-0.5,0,0), new THREE.Vector3(0.5,0,0)
  ]), MAT_COTA_LINEA);
  const flechaLargoA = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA);
  const flechaLargoB = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA);
  const lblLargo = crearSpriteCota();
  const lineaAncho = new THREE.Line(new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(0,0,-0.5), new THREE.Vector3(0,0,0.5)
  ]), MAT_COTA_LINEA);
  const flechaAnchoA = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA);
  const flechaAnchoB = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA);
  const lblAncho = crearSpriteCota();
  const lineaAlto = new THREE.Line(new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(0,-0.5,0), new THREE.Vector3(0,0.5,0)
  ]), MAT_COTA_LINEA);
  const flechaAltoA = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA);
  const flechaAltoB = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA);
  const lblAlto = crearSpriteCota();
  g.add(lineaLargo, flechaLargoA, flechaLargoB, lblLargo);
  g.add(lineaAncho, flechaAnchoA, flechaAnchoB, lblAncho);
  g.add(lineaAlto, flechaAltoA, flechaAltoB, lblAlto);
  g.renderOrder = 998;
  return {
    grupo: g,
    largo: { line: lineaLargo, a: flechaLargoA, b: flechaLargoB, lbl: lblLargo },
    ancho: { line: lineaAncho, a: flechaAnchoA, b: flechaAnchoB, lbl: lblAncho },
    alto:  { line: lineaAlto, a: flechaAltoA, b: flechaAltoB, lbl: lblAlto }
  };
}
ORDEN_UI.forEach(key => {
  const c = crearCota();
  c.grupo.visible = false;
  grupoRaiz.add(c.grupo);
  cotas[key] = c;
});

function actualizarCotas(key){
  const def = ELEMENTOS[key];
  const c = cotas[key];
  const { largo:L, ancho:A, alto:H } = def.dims;
  const p = def.pos;
  const off = 0.30;
  const u = def.unidades || { largo:'m', ancho:'m', alto:'m' };

  const yL = p.y + 0.02, zL = p.z - A/2 - off;
  const vL1 = new THREE.Vector3(p.x - L/2, yL, zL);
  const vL2 = new THREE.Vector3(p.x + L/2, yL, zL);
  c.largo.line.geometry.setFromPoints([vL1, vL2]);
  c.largo.a.position.copy(vL1); c.largo.a.rotation.set(0, 0, Math.PI/2);
  c.largo.b.position.copy(vL2); c.largo.b.rotation.set(0, 0, -Math.PI/2);
  c.largo.lbl.position.set(p.x, yL + 0.16, zL);
  dibujarCota(c.largo.lbl, formatearMedida(L, u.largo));

  const yA = p.y + 0.02, xA = p.x + L/2 + off;
  const vA1 = new THREE.Vector3(xA, yA, p.z - A/2);
  const vA2 = new THREE.Vector3(xA, yA, p.z + A/2);
  c.ancho.line.geometry.setFromPoints([vA1, vA2]);
  c.ancho.a.position.copy(vA1); c.ancho.a.rotation.set(-Math.PI/2, 0, 0);
  c.ancho.b.position.copy(vA2); c.ancho.b.rotation.set(Math.PI/2, 0, 0);
  c.ancho.lbl.position.set(xA + 0.18, yA + 0.16, p.z);
  dibujarCota(c.ancho.lbl, formatearMedida(A, u.ancho));

  const xH = p.x - L/2 - off, zH = p.z - A/2 - off;
  const vH1 = new THREE.Vector3(xH, p.y, zH);
  const vH2 = new THREE.Vector3(xH, p.y + H, zH);
  c.alto.line.geometry.setFromPoints([vH1, vH2]);
  c.alto.a.position.copy(vH1); c.alto.a.rotation.set(Math.PI, 0, 0);
  c.alto.b.position.copy(vH2); c.alto.b.rotation.set(0, 0, 0);
  c.alto.lbl.position.set(xH - 0.22, p.y + H/2, zH);
  dibujarCota(c.alto.lbl, formatearMedida(H, u.alto));
}

const cotasVano = (() => {
  const g = new THREE.Group();
  const lineA = new THREE.Line(new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(-0.5,0,0), new THREE.Vector3(0.5,0,0)
  ]), MAT_COTA_VANO_LINEA);
  const fA1 = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA_VANO);
  const fA2 = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA_VANO);
  const lblA = crearSpriteCota('rgba(30,64,175,0.96)');
  const lineB = new THREE.Line(new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(0,-0.5,0), new THREE.Vector3(0,0.5,0)
  ]), MAT_COTA_VANO_LINEA);
  const fB1 = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA_VANO);
  const fB2 = new THREE.Mesh(GEO_FLECHA, MAT_FLECHA_VANO);
  const lblB = crearSpriteCota('rgba(30,64,175,0.96)');
  g.add(lineA, fA1, fA2, lblA);
  g.add(lineB, fB1, fB2, lblB);
  g.visible = false;
  g.renderOrder = 998;
  grupoRaiz.add(g);
  return {
    grupo: g,
    ancho: { line: lineA, a: fA1, b: fA2, lbl: lblA },
    alto:  { line: lineB, a: fB1, b: fB2, lbl: lblB }
  };
})();

function actualizarCotasVano(){
  const m = ELEMENTOS.muro;
  const { vano } = m;
  const A = m.dims.ancho;
  const uv = m.unidadesVano || { ancho:'cm', alto:'cm' };
  const yCentro = m.pos.y + vano.alfeizar + vano.alto/2;
  const zFrente = m.pos.z + A/2 + 0.001;
  const off = 0.18;

  const yA = yCentro - vano.alto/2 - off;
  const vA1 = new THREE.Vector3(m.pos.x - vano.ancho/2, yA, zFrente);
  const vA2 = new THREE.Vector3(m.pos.x + vano.ancho/2, yA, zFrente);
  cotasVano.ancho.line.geometry.setFromPoints([vA1, vA2]);
  cotasVano.ancho.a.position.copy(vA1); cotasVano.ancho.a.rotation.set(0, 0, Math.PI/2);
  cotasVano.ancho.b.position.copy(vA2); cotasVano.ancho.b.rotation.set(0, 0, -Math.PI/2);
  cotasVano.ancho.lbl.position.set(m.pos.x, yA - 0.14, zFrente);
  dibujarCota(cotasVano.ancho.lbl, formatearMedida(vano.ancho, uv.ancho));

  const xB = m.pos.x + vano.ancho/2 + off;
  const vB1 = new THREE.Vector3(xB, yCentro - vano.alto/2, zFrente);
  const vB2 = new THREE.Vector3(xB, yCentro + vano.alto/2, zFrente);
  cotasVano.alto.line.geometry.setFromPoints([vB1, vB2]);
  cotasVano.alto.a.position.copy(vB1); cotasVano.alto.a.rotation.set(Math.PI, 0, 0);
  cotasVano.alto.b.position.copy(vB2); cotasVano.alto.b.rotation.set(0, 0, 0);
  cotasVano.alto.lbl.position.set(xB + 0.18, yCentro, zFrente);
  dibujarCota(cotasVano.alto.lbl, formatearMedida(vano.alto, uv.alto));
}

function mostrarCotasDe(key){
  for (const k of ORDEN_UI) cotas[k].grupo.visible = (k === key);
  cotasVano.grupo.visible = (key === 'muro');
  if (key) actualizarCotas(key);
  if (key === 'muro') actualizarCotasVano();
}

/* =========================================================
   7. GEOMETRÍA Y OPACIDADES
   ========================================================= */
function actualizarOpacidades(){
  for (const key of ORDEN_UI){
    const def = ELEMENTOS[key];
    const o = objetos[key];
    const activo = (key === selKey);
    const oActiva   = def.opacidadActiva   ?? 1.0;
    const oInactiva = def.opacidadInactiva ?? 0.72;
    const op = activo ? oActiva : oInactiva;
    o.molde.material.transparent = op < 1;
    o.molde.material.opacity = op;
    o.molde.material.needsUpdate = true;
    o.aristas.material.opacity = activo ? 1.0 : 0.55;
  }
}
function actualizarGeometria(key){
  const def = ELEMENTOS[key];
  const o = objetos[key];
  const { largo:L, ancho:A, alto:H } = def.dims;
  const p = def.pos;
  o.molde.scale.set(L, H, A);
  o.molde.position.set(p.x, p.y + H / 2, p.z);
  o.aristas.scale.copy(o.molde.scale);
  o.aristas.position.copy(o.molde.position);
  o.etiqueta.position.set(p.x, 0, p.z);
  fijarRelleno(key, o.fillH);
}
function actualizarEscalaEtiquetas(){
  const escalaX = frustum * 0.20, escalaY = frustum * 0.05, gap = escalaY * 0.35;
  for (const key of ORDEN_UI){
    const o = objetos[key], def = ELEMENTOS[key];
    o.etiqueta.scale.set(escalaX, escalaY, 1);
    o.etiqueta.position.y = def.pos.y + def.dims.alto + escalaY / 2 + gap;
  }
  const cotaX = Math.max(0.66, frustum * 0.105);
  const cotaY = Math.max(0.18, frustum * 0.028);
  for (const key of ORDEN_UI){
    const c = cotas[key];
    c.largo.lbl.scale.set(cotaX, cotaY, 1);
    c.ancho.lbl.scale.set(cotaX, cotaY, 1);
    c.alto.lbl.scale.set(cotaX, cotaY, 1);
  }
  cotasVano.ancho.lbl.scale.set(cotaX, cotaY, 1);
  cotasVano.alto.lbl.scale.set(cotaX, cotaY, 1);
}
function actualizarTodo(){
  ORDEN_UI.forEach(actualizarGeometria);
  actualizarEtiquetas();
  actualizarEscalaEtiquetas();
  actualizarOpacidades();
  actualizarVano();
  actualizarCotas(selKey);
}
function actualizarEtiquetas(){
  for (const key of ORDEN_UI){
    const def = ELEMENTOS[key], o = objetos[key];
    dibujarEtiqueta(o.etiqueta, def.corto, key === selKey);
  }
}

/* =========================================================
   8. CHARCO Y FLUJO
   ========================================================= */
const geoCharco = new THREE.CircleGeometry(1, 48);
(function deformar(){
  const pos = geoCharco.attributes.position;
  for (let i = 0; i < pos.count; i++){
    const x = pos.getX(i), y = pos.getY(i);
    const r = Math.hypot(x, y);
    if (r > 0.01){
      const a = Math.atan2(y, x);
      const n = 1 + (Math.random() - 0.5) * 0.35;
      pos.setX(i, Math.cos(a) * r * n);
      pos.setY(i, Math.sin(a) * r * n);
    }
  }
  geoCharco.computeVertexNormals();
})();
const charcoMat = new THREE.MeshStandardMaterial({
  color: 0x6b7280, roughness: 0.85, metalness: 0.02, transparent: true, opacity: 0
});
const charco = new THREE.Mesh(geoCharco, charcoMat);
charco.rotation.x = -Math.PI / 2;
charco.position.y = 0.020;
charco.visible = false;
charco.renderOrder = 2;
scene.add(charco);
const charcoBorde = new THREE.LineSegments(new THREE.EdgesGeometry(geoCharco),
  new THREE.LineBasicMaterial({ color: 0x374151, transparent: true, opacity: 0 }));
charcoBorde.rotation.x = -Math.PI / 2;
charcoBorde.position.y = 0.022;
charcoBorde.visible = false;
charcoBorde.renderOrder = 3;
scene.add(charcoBorde);

function mostrarCharco(def, progreso, exceso){
  const rBase = Math.max(def.dims.largo, def.dims.ancho) * 0.55;
  const rMax = rBase * (1 + exceso * 1.6);
  const r = rMax * progreso;
  charco.visible = true; charcoBorde.visible = true;
  charco.scale.set(r, r, 1); charcoBorde.scale.set(r, r, 1);
  charco.position.x = def.pos.x; charco.position.z = def.pos.z;
  charcoBorde.position.x = def.pos.x; charcoBorde.position.z = def.pos.z;
  const op = Math.min(1, progreso) * 0.95;
  charcoMat.opacity = op;
  charcoBorde.material.opacity = op * 0.85;
}
function ocultarCharco(){
  charco.visible = false; charcoBorde.visible = false;
  charcoMat.opacity = 0; charcoBorde.material.opacity = 0;
}

function crearTexturaLiquida(){
  const cv = document.createElement('canvas');
  cv.width = 32; cv.height = 64;
  const ctx = cv.getContext('2d');
  const g = ctx.createLinearGradient(0, 0, 0, 64);
  g.addColorStop(0.00, '#b8bec5'); g.addColorStop(0.35, '#9aa1a8');
  g.addColorStop(0.55, '#c4cad0'); g.addColorStop(1.00, '#8f969d');
  ctx.fillStyle = g; ctx.fillRect(0, 0, 32, 64);
  for (let i = 0; i < 26; i++){
    ctx.fillStyle = `rgba(255,255,255,${0.04 + Math.random() * 0.09})`;
    ctx.fillRect(Math.random()*32, Math.random()*64, 2+Math.random()*5, 1+Math.random()*3);
  }
  const tex = new THREE.CanvasTexture(cv);
  tex.wrapS = tex.wrapT = THREE.RepeatWrapping;
  tex.repeat.set(1, 5);
  tex.colorSpace = THREE.SRGBColorSpace;
  return tex;
}
const texturaFlujo = crearTexturaLiquida();
const flujo = new THREE.Mesh(
  new THREE.CylinderGeometry(0.075, 0.105, 1, 14, 1, true),
  new THREE.MeshStandardMaterial({
    map: texturaFlujo, color: 0xa8aeb5, roughness: 0.55, metalness: 0.05,
    side: THREE.DoubleSide
  })
);
flujo.visible = false; flujo.renderOrder = 5;
scene.add(flujo);
const tolva = new THREE.Mesh(
  new THREE.CylinderGeometry(0.34, 0.14, 0.5, 16, 1, true),
  new THREE.MeshStandardMaterial({ color:0x94a3b8, roughness:0.6, metalness:0.25,
    side:THREE.DoubleSide })
);
tolva.visible = false;
scene.add(tolva);
function mostrarFlujo(def, alturaHormigon){
  const H = def.dims.alto;
  const sup = def.pos.y + alturaHormigon;
  const top = def.pos.y + H + 2.1;
  const len = Math.max(0.08, top - sup);
  flujo.visible = true;
  flujo.scale.set(1, len, 1);
  flujo.position.set(def.pos.x, (top + sup) / 2, def.pos.z);
  tolva.visible = true;
  tolva.position.set(def.pos.x, top + 0.25, def.pos.z);
}
function ocultarFlujo(){ flujo.visible = false; tolva.visible = false; }

/* =========================================================
   9. PARTÍCULAS
   ========================================================= */
const PCOUNT = 600;
const pPos = new Float32Array(PCOUNT * 3);
const pVel = new Float32Array(PCOUNT * 3);
const pLife = new Float32Array(PCOUNT);
for (let i = 0; i < PCOUNT; i++) pPos[i * 3 + 1] = -999;
const pGeo = new THREE.BufferGeometry();
pGeo.setAttribute('position', new THREE.BufferAttribute(pPos, 3));
const puntos = new THREE.Points(pGeo, new THREE.PointsMaterial({
  color: 0x7a828a, size: 0.09, sizeAttenuation: true, transparent: true, opacity: 0.95
}));
puntos.frustumCulled = false;
scene.add(puntos);
let pCursor = 0;
function spawnParticula(x, y, z, vx, vy, vz){
  for (let n = 0; n < PCOUNT; n++){
    const i = (pCursor + n) % PCOUNT;
    if (pLife[i] <= 0){
      pCursor = (i + 1) % PCOUNT;
      pPos[i*3] = x; pPos[i*3+1] = y; pPos[i*3+2] = z;
      pVel[i*3] = vx; pVel[i*3+1] = vy; pVel[i*3+2] = vz;
      pLife[i] = 0.7 + Math.random() * 0.9;
      return;
    }
  }
}
function spawnDesborde(def){
  const { largo:L, ancho:A, alto:H } = def.dims;
  const y = def.pos.y + H + 0.02;
  const lado = Math.floor(Math.random() * 4);
  const sp = 0.6 + Math.random() * 1.1;
  let x, z, vx, vz;
  if (lado === 0){ x=(Math.random()-0.5)*L; z= A/2; vx=(Math.random()-0.5)*0.6; vz= sp; }
  else if (lado === 1){ x=(Math.random()-0.5)*L; z=-A/2; vx=(Math.random()-0.5)*0.6; vz=-sp; }
  else if (lado === 2){ x= L/2; z=(Math.random()-0.5)*A; vx= sp; vz=(Math.random()-0.5)*0.6; }
  else { x=-L/2; z=(Math.random()-0.5)*A; vx=-sp; vz=(Math.random()-0.5)*0.6; }
  spawnParticula(def.pos.x + x, y, def.pos.z + z, vx, 0.6 + Math.random()*1.2, vz);
}
function spawnGoterones(def){
  const { largo:L, ancho:A, alto:H } = def.dims;
  const yTop = def.pos.y + H + 0.05;
  const n = 2 + Math.floor(Math.random() * 3);
  for (let i = 0; i < n; i++){
    const lado = Math.floor(Math.random() * 4);
    const sep = 0.35 + Math.random() * 0.6;
    let x, z, vx, vz;
    if (lado === 0){ x=(Math.random()-0.5)*L; z= A/2; vx=(Math.random()-0.5)*0.2; vz= sep; }
    else if (lado === 1){ x=(Math.random()-0.5)*L; z=-A/2; vx=(Math.random()-0.5)*0.2; vz=-sep; }
    else if (lado === 2){ x= L/2; z=(Math.random()-0.5)*A; vx= sep; vz=(Math.random()-0.5)*0.2; }
    else { x=-L/2; z=(Math.random()-0.5)*A; vx=-sep; vz=(Math.random()-0.5)*0.2; }
    spawnParticula(def.pos.x + x, yTop, def.pos.z + z, vx, -0.4, vz);
  }
}
function actualizarParticulas(dt){
  let dirty = false;
  for (let i = 0; i < PCOUNT; i++){
    if (pLife[i] <= 0) continue;
    pLife[i] -= dt;
    if (pLife[i] <= 0){ pPos[i*3+1] = -999; dirty = true; continue; }
    pVel[i*3+1] -= 9.8 * dt;
    pPos[i*3] += pVel[i*3] * dt;
    pPos[i*3+1] += pVel[i*3+1] * dt;
    pPos[i*3+2] += pVel[i*3+2] * dt;
    if (pPos[i*3+1] < 0.03){
      pPos[i*3+1] = 0.03;
      pVel[i*3+1] *= -0.25; pVel[i*3] *= 0.5; pVel[i*3+2] *= 0.5;
    }
    dirty = true;
  }
  if (dirty) pGeo.attributes.position.needsUpdate = true;
}

/* =========================================================
   10. RELLENO / VOLUMEN
   ========================================================= */
function fijarRelleno(key, h){
  const def = ELEMENTOS[key], o = objetos[key];
  const H = def.dims.alto;
  const hh = Math.max(0, Math.min(h, H));
  o.fillH = hh;
  if (hh < 0.0008){ o.concreto.visible = false; return; }
  o.concreto.visible = true;
  o.concreto.scale.set(def.dims.largo, hh, def.dims.ancho);
  o.concreto.position.set(def.pos.x, def.pos.y + hh / 2, def.pos.z);
}
function limpiarRellenos(){
  for (const k of ORDEN_UI){ objetos[k].fillH = 0; objetos[k].concreto.visible = false; }
  ocultarFlujo(); ocultarCharco();
  for (let i = 0; i < PCOUNT; i++){ pLife[i] = 0; pPos[i*3+1] = -999; }
  pGeo.attributes.position.needsUpdate = true;
}
function volumenReal(key){
  const def = ELEMENTOS[key];
  const { largo:L, ancho:A, alto:H } = def.dims;
  let v = L * A * H;
  if (key === 'muro' && def.vano){
    v -= def.vano.ancho * def.vano.alto * A;
    v = Math.max(v, 0.001);
  }
  return v;
}

/* =========================================================
   11. PANEL DE COSTO ACUMULATIVO
   ========================================================= */
function actualizarPanelCostos(){
  let volRealAcum = 0, volEstAcum = 0, cubicados = 0;
  for (const k of ORDEN_UI){
    const e = estadoProyecto[k];
    if (e){ volRealAcum += e.real; volEstAcum += e.estimado; cubicados++; }
  }
  const costoReal = volRealAcum * PRECIO_HORMIGON_CLP;
  const costoEst  = volEstAcum * PRECIO_HORMIGON_CLP;
  const difMonetaria = costoEst - costoReal;
  const difVolumen = volEstAcum - volRealAcum;

  document.getElementById('cost-prog').textContent = `${cubicados} / ${ORDEN_UI.length}`;
  document.getElementById('cost-vol-real').innerHTML = volRealAcum.toFixed(3) + ' <small>m³</small>';
  document.getElementById('cost-vol-est').innerHTML  = volEstAcum.toFixed(3) + ' <small>m³</small>';
  document.getElementById('cost-real').textContent = formatearCLP(costoReal);
  document.getElementById('cost-est').textContent  = formatearCLP(costoEst);

  const lossRow = document.getElementById('cost-loss-row');
  const saveRow = document.getElementById('cost-save-row');
  const balRow  = document.getElementById('cost-balance-row');

  if (difVolumen > 0.0005){
    document.getElementById('cost-loss').textContent = formatearCLP(difMonetaria);
    lossRow.style.display = 'flex'; saveRow.style.display = 'none';
  } else if (difVolumen < -0.0005){
    document.getElementById('cost-save').textContent = formatearCLP(Math.abs(difMonetaria));
    saveRow.style.display = 'flex'; lossRow.style.display = 'none';
  } else {
    lossRow.style.display = 'none'; saveRow.style.display = 'none';
  }

  if (cubicados === ORDEN_UI.length){
    balRow.style.display = 'flex';
    const bal = document.getElementById('cost-balance');
    if (difVolumen > 0.0005){
      bal.textContent = '+ ' + formatearCLP(difMonetaria) + ' (exceso)';
      bal.style.color = '#b91c1c'; balRow.style.background = '#fef2f2';
    } else if (difVolumen < -0.0005){
      bal.textContent = '- ' + formatearCLP(Math.abs(difMonetaria)) + ' (déficit)';
      bal.style.color = '#b45309'; balRow.style.background = '#fff7ed';
    } else {
      bal.textContent = '$0 (exacto)';
      bal.style.color = '#15803d'; balRow.style.background = '#ecfdf5';
    }
  } else {
    balRow.style.display = 'none';
  }

  /* Mostrar/ocultar botón del informe */
  const btnInf = document.getElementById('btn-informe');
  if (btnInf) btnInf.style.display = cubicados > 0 ? 'block' : 'none';
}
function limpiarPanelCostos(){
  document.getElementById('cost-prog').textContent = `0 / ${ORDEN_UI.length}`;
  document.getElementById('cost-vol-real').innerHTML = '0.000 <small>m³</small>';
  document.getElementById('cost-vol-est').innerHTML  = '0.000 <small>m³</small>';
  document.getElementById('cost-real').textContent = '$0';
  document.getElementById('cost-est').textContent  = '$0';
  document.getElementById('cost-loss-row').style.display = 'none';
  document.getElementById('cost-save-row').style.display = 'none';
  document.getElementById('cost-balance-row').style.display = 'none';
  const btnInf = document.getElementById('btn-informe');
  if (btnInf) btnInf.style.display = 'none';
}

/* =========================================================
   11b. INFORME FINAL DE CUBICACIONES
   ========================================================= */
function generarInformeHTML(){
  const fecha = new Date().toLocaleString('es-CL', {
    day:'2-digit', month:'2-digit', year:'numeric',
    hour:'2-digit', minute:'2-digit'
  });

  let volRealAcum = 0, volEstAcum = 0, costoRealAcum = 0, costoEstAcum = 0;
  let nExactos = 0, nFalta = 0, nDesborda = 0;
  let filas = '';

  for (const k of ORDEN_UI){
    const e = estadoProyecto[k];
    const def = ELEMENTOS[k];

    if (!e){
      filas += `<tr style="opacity:.5">
        <td><b>${def.corto}</b><br><span class="dim-small">${def.nombre}</span></td>
        <td colspan="8" style="text-align:center;font-style:italic;color:#94a3b8">
          Pendiente de cubicar
        </td></tr>`;
      continue;
    }

    volRealAcum += e.real;
    volEstAcum  += e.estimado;
    const costoReal = e.real * PRECIO_HORMIGON_CLP;
    const costoEst  = e.estimado * PRECIO_HORMIGON_CLP;
    costoRealAcum += costoReal;
    costoEstAcum  += costoEst;

    if (e.cls === 'ok')        nExactos++;
    else if (e.cls === 'warn') nFalta++;
    else                       nDesborda++;

    const color = e.cls === 'ok' ? '#15803d'
                : e.cls === 'warn' ? '#c2410c'
                : '#b91c1c';
    const icono = e.cls === 'ok' ? '✓'
                : e.cls === 'warn' ? '▼'
                : '▲';

    const uD = e.dimsUsuario || e.dims;
    const rD = e.dims;

    filas += `<tr>
      <td><b>${def.corto}</b><br><span class="dim-small">${def.nombre}</span></td>
      <td class="mono">${uD.L.toFixed(3)} × ${uD.A.toFixed(3)} × ${uD.H.toFixed(3)}</td>
      <td class="mono">${rD.L.toFixed(3)} × ${rD.A.toFixed(3)} × ${rD.H.toFixed(3)}</td>
      <td class="mono">${e.estimado.toFixed(3)}</td>
      <td class="mono">${e.real.toFixed(3)}</td>
      <td class="mono" style="color:${color};font-weight:800">
        ${icono} ${(e.diff>=0?'+':'')}${e.diff.toFixed(3)}</td>
      <td class="mono" style="color:${color}">${(e.pct>=0?'+':'')}${e.pct.toFixed(2)}%</td>
      <td class="mono">${formatearCLP(costoEst)}</td>
      <td class="mono">${formatearCLP(costoReal)}</td>
    </tr>`;
  }

  const difVol = volEstAcum - volRealAcum;
  const difMon = costoEstAcum - costoRealAcum;
  const totalColor = Math.abs(difVol) < 0.0005 ? '#15803d'
                    : difVol > 0 ? '#b91c1c' : '#c2410c';
  const balanceTxt = Math.abs(difVol) < 0.0005 ? '$0 (exacto)'
                    : difVol > 0 ? '+ ' + formatearCLP(difMon) + ' (sobrecosto)'
                    : '− ' + formatearCLP(Math.abs(difMon)) + ' (ahorro)';
  const precision = juego.intentos > 0
    ? (juego.aciertos / juego.intentos * 100).toFixed(1) : '0.0';

  const impacto = Math.abs(difVol) < 0.0005
    ? 'El proyecto se ajusta exactamente al volumen real requerido, sin sobrecostos ni déficit de material.'
    : difVol > 0
      ? `Existe un <b>sobrecosto de ${formatearCLP(difMon)}</b> por exceso de hormigón (${difVol.toFixed(3)} m³ adicionales). En obra esto implica desperdicio de material y mayor costo directo.`
      : `Existe un <b>déficit de ${Math.abs(difVol).toFixed(3)} m³</b> (${formatearCLP(Math.abs(difMon))} por debajo del presupuesto real). En obra esto obligaría a detener la faena y solicitar un camión adicional, con el consiguiente retraso.`;

  return `
    <div class="inf-header">
      <div>
        <div class="inf-eyebrow">INFORME TÉCNICO DE CUBICACIÓN</div>
        <h1 class="inf-title">Cubicación de Hormigón Estructural</h1>
        <p class="inf-sub">Proyecto de fundación, muro, pilar y losa · Hormigón G25</p>
      </div>
      <div class="inf-meta">
        <div><span class="lbl">Fecha</span><br><b>${fecha}</b></div>
        <div style="margin-top:6px"><span class="lbl">Precio unitario</span><br>
          <b>${formatearCLP(PRECIO_HORMIGON_CLP)} / m³</b></div>
      </div>
    </div>

    <div class="inf-body">
      <section class="inf-section">
        <h2>1 · Resumen ejecutivo</h2>
        <div class="inf-kpis">
          <div class="kpi"><div class="kpi-lbl">Elementos cubicados</div>
            <div class="kpi-val">${juego.elementosCubicados} / ${ORDEN_UI.length}</div></div>
          <div class="kpi"><div class="kpi-lbl">Volumen real</div>
            <div class="kpi-val">${volRealAcum.toFixed(3)} <small>m³</small></div></div>
          <div class="kpi"><div class="kpi-lbl">Volumen cubicado</div>
            <div class="kpi-val">${volEstAcum.toFixed(3)} <small>m³</small></div></div>
          <div class="kpi"><div class="kpi-lbl">Costo real</div>
            <div class="kpi-val">${formatearCLP(costoRealAcum)}</div></div>
          <div class="kpi"><div class="kpi-lbl">Costo cubicado</div>
            <div class="kpi-val">${formatearCLP(costoEstAcum)}</div></div>
          <div class="kpi" style="border-color:${totalColor};background:${totalColor}14">
            <div class="kpi-lbl">Balance final</div>
            <div class="kpi-val" style="color:${totalColor}">${balanceTxt}</div></div>
        </div>
      </section>

      <section class="inf-section">
        <h2>2 · Detalle por elemento estructural</h2>
        <table class="inf-tabla">
          <thead>
            <tr>
              <th rowspan="2">Elemento</th>
              <th colspan="2">Dimensiones (m) · L × A × H</th>
              <th colspan="3">Volumen (m³)</th>
              <th rowspan="2">Error<br>relativo</th>
              <th colspan="2">Costo hormigón G25</th>
            </tr>
            <tr>
              <th>Ingresadas por el estudiante</th>
              <th>Cotas reales del modelo</th>
              <th>Estimado</th><th>Real</th><th>Diferencia</th>
              <th>Estimado</th><th>Real</th>
            </tr>
          </thead>
          <tbody>${filas}</tbody>
          <tfoot>
            <tr>
              <td colspan="3" style="text-align:right"><b>TOTALES</b></td>
              <td class="mono"><b>${volEstAcum.toFixed(3)}</b></td>
              <td class="mono"><b>${volRealAcum.toFixed(3)}</b></td>
              <td class="mono" style="color:${totalColor}"><b>${(difVol>=0?'+':'')}${difVol.toFixed(3)}</b></td>
              <td class="mono" style="color:${totalColor}"><b>${
                volRealAcum>0 ? ((difVol/volRealAcum*100).toFixed(2)) : '0.00'}%</b></td>
              <td class="mono"><b>${formatearCLP(costoEstAcum)}</b></td>
              <td class="mono"><b>${formatearCLP(costoRealAcum)}</b></td>
            </tr>
          </tfoot>
        </table>
      </section>

      <section class="inf-section">
        <h2>3 · Desempeño del estudiante</h2>
        <div class="inf-desempeno">
          <div class="desemp-item"><div class="desemp-lbl">Puntos totales</div>
            <div class="desemp-val" style="color:#1d4ed8">${juego.puntos}</div></div>
          <div class="desemp-item"><div class="desemp-lbl">Aciertos exactos</div>
            <div class="desemp-val" style="color:#15803d">${juego.aciertos} / ${juego.intentos}</div></div>
          <div class="desemp-item"><div class="desemp-lbl">Precisión global</div>
            <div class="desemp-val" style="color:#7c3aed">${precision}%</div></div>
          <div class="desemp-item"><div class="desemp-lbl">Estrellas</div>
            <div class="desemp-val" style="color:#b45309">${juego.estrellasTotales} ⭐</div></div>
          <div class="desemp-item"><div class="desemp-lbl">Mejor racha</div>
            <div class="desemp-val" style="color:#dc2626">${juego.mejorRacha}</div></div>
        </div>
        <div class="inf-desglose">
          <span class="chip ok">✓ Exactos: ${nExactos}</span>
          <span class="chip warn">▼ Faltantes: ${nFalta}</span>
          <span class="chip bad">▲ Excedidos: ${nDesborda}</span>
        </div>
      </section>

      <section class="inf-section">
        <h2>4 · Observaciones técnicas</h2>
        <div class="inf-obs">
          <p><b>Metodología:</b> El volumen de cada elemento se calcula como V = L × A × H,
          con las dimensiones convertidas a metros. Para el muro se descuenta el volumen del
          vano (ventana) según las cotas indicadas en el modelo 3D. Se consideran además el
          emplantillado y el cimiento como base continua bajo muro y pilar.</p>
          <p><b>Criterio de aceptación:</b> Se considera cubicación exacta aquella cuyo error
          relativo respecto al volumen real es ≤ 2 %.</p>
          <p><b>Impacto económico:</b> ${impacto}</p>
        </div>
      </section>

      <div class="inf-firma">
        <div class="firma-linea"></div>
        <div class="firma-lbl">Firma del estudiante</div>
      </div>
    </div>

    <div class="inf-acciones">
      <button class="inf-btn secundario" id="inf-cerrar">Cerrar</button>
      <button class="inf-btn primario" id="inf-print">🖨️ Imprimir / Guardar PDF</button>
    </div>
  `;
}

function mostrarInforme(){
  document.getElementById('informe-contenido').innerHTML = generarInformeHTML();
  document.getElementById('informe-final').classList.add('show');
  document.getElementById('inf-print')?.addEventListener('click',
    () => { sonidoClick(); window.print(); });
  document.getElementById('inf-cerrar')?.addEventListener('click',
    () => { sonidoClick(); ocultarInforme(); });
}
function ocultarInforme(){
  document.getElementById('informe-final').classList.remove('show');
}

/* =========================================================
   12. VALIDACIÓN DE DIMENSIONES
   ========================================================= */
function validarDimensiones(){
  const def = ELEMENTOS[selKey];
  const L = parseFloat(inLargo.value);
  const A = parseFloat(inAncho.value);
  const H = parseFloat(inAlto.value);
  const rL = def.dims.largo, rA = def.dims.ancho, rH = def.dims.alto;
  const tol = 0.02;
  const errores = [];
  const chk = (val, real, dim, campo, unidad) => {
    const ok = isFinite(val) && Math.abs(val - real) / real <= tol;
    campo.classList.remove('ok','error');
    campo.classList.add(ok ? 'ok' : 'error');
    if (!ok) errores.push({ dim, val, real, unidad });
  };
  chk(L, rL, 'largo', inLargo, def.unidades?.largo || 'm');
  chk(A, rA, 'ancho', inAncho, def.unidades?.ancho || 'm');
  chk(H, rH, 'alto',  inAlto,  def.unidades?.alto  || 'm');
  return errores;
}
function mensajeErrorDimensiones(errores){
  const nombres = { largo:'Largo', ancho:'Ancho', alto:'Alto' };
  return errores.map(e => {
    let pista = '';
    if (e.unidad === 'cm') pista = 'Recuerda: 1 cm = 0.01 m.';
    if (e.unidad === 'mm') pista = 'Recuerda: 1 mm = 0.001 m.';
    return `<b>${nombres[e.dim]}</b>: tu valor está fuera de rango. ${pista}`;
  }).join('<br>');
}

/* =========================================================
   13. BLOQUEO
   ========================================================= */
function bloquearElemento(key, cls){
  const btn = document.querySelector(`.elem-btn[data-key="${key}"]`);
  if (!btn) return;
  btn.classList.add('cubicado');
  btn.classList.remove('ok','warn','bad');
  btn.classList.add(cls);
}
function desbloquearTodos(){
  document.querySelectorAll('.elem-btn').forEach(b =>
    b.classList.remove('cubicado','ok','warn','bad'));
}
function restaurarResultadoElemento(key){
  const e = estadoProyecto[key];
  if (!e) return;
  inLargo.value = e.dims.L.toFixed(3);
  inAncho.value = e.dims.A.toFixed(3);
  inAlto.value  = e.dims.H.toFixed(3);
  inVol.value   = e.estimado.toFixed(3);
  actualizarFormulaViva();
  [inLargo, inAncho, inAlto].forEach(i => i.classList.add('ok'));
  badge.className = 'badge ' + e.cls; badge.textContent = e.txt;
  rReal.textContent = e.real.toFixed(3) + ' m³';
  rEst.textContent  = e.estimado.toFixed(3) + ' m³';
  rDif.textContent  = (e.diff >= 0 ? '+' : '') + e.diff.toFixed(3) + ' m³';
  rPct.textContent  = (e.pct >= 0 ? '+' : '') + e.pct.toFixed(2) + ' %';
  rDif.className = 'v ' + e.cls; rPct.className = 'v ' + e.cls;
  barraFill.style.width = (Math.min(e.ratio, 1.6) / 1.6 * 100).toFixed(1) + '%';
  barraFill.className = e.cls;
  pctLlenado.textContent = (e.ratio * 100).toFixed(1) + ' %';
  feedback.className = 'feedback ' + e.cls;
  feedback.innerHTML = e.mensaje;
}

/* =========================================================
   14. CÁMARA
   ========================================================= */
function centrarEnElemento(key, suave = true){
  const def = ELEMENTOS[key];
  const { largo:L, ancho:A, alto:H } = def.dims;
  const p = def.pos;
  const maxDim = Math.max(L, A, H);
  const nuevoFrustum = Math.max(2.4, maxDim * 1.8);
  camGoal.target.set(p.x, p.y + H / 2, p.z);
  camGoal.frustum = nuevoFrustum;
  camLerp = suave ? 0 : 1;
  if (!suave){
    camCurrent.target.copy(camGoal.target);
    camCurrent.frustum = camGoal.frustum;
    frustum = camCurrent.frustum;
    actualizarCamara(); redimensionar();
  }
}
function actualizarTransicionCamara(dt){
  if (camLerp >= 1) return;
  camLerp = Math.min(1, camLerp + dt * 2.0);
  const k = camLerp * camLerp * (3 - 2 * camLerp);
  camCurrent.target.x = THREE.MathUtils.lerp(camCurrent.target.x, camGoal.target.x, k);
  camCurrent.target.y = THREE.MathUtils.lerp(camCurrent.target.y, camGoal.target.y, k);
  camCurrent.target.z = THREE.MathUtils.lerp(camCurrent.target.z, camGoal.target.z, k);
  frustum = THREE.MathUtils.lerp(frustum, camGoal.frustum, k);
  camCurrent.frustum = frustum;
  actualizarCamara(); redimensionar();
}

/* =========================================================
   15. UI
   ========================================================= */
const selector   = document.getElementById('selector');
const inLargo    = document.getElementById('in-largo');
const inAncho    = document.getElementById('in-ancho');
const inAlto     = document.getElementById('in-alto');
const inVol      = document.getElementById('in-vol');
const lblLargo   = document.getElementById('lbl-largo');
const lblAncho   = document.getElementById('lbl-ancho');
const lblAlto    = document.getElementById('lbl-alto');
const badge      = document.getElementById('badge');
const rReal      = document.getElementById('r-real');
const rEst       = document.getElementById('r-est');
const rDif       = document.getElementById('r-dif');
const rPct       = document.getElementById('r-pct');
const barraFill  = document.getElementById('barra-fill');
const pctLlenado = document.getElementById('pct-llenado');
const feedback   = document.getElementById('feedback');
const hudElem    = document.getElementById('hud-elem');
const hudDims    = document.getElementById('hud-dims');
const leyenda    = document.getElementById('leyenda');
const formulaViva = document.getElementById('formula-viva');
const avisoDim   = document.getElementById('aviso-dim');
const notaDetalle = document.getElementById('nota-detalle');
const notaMuro    = document.getElementById('nota-muro');
const notaPilar   = document.getElementById('nota-pilar');

let selKey = 'muro';

ORDEN_UI.forEach((key, i) => {
  const def = ELEMENTOS[key];
  const btn = document.createElement('button');
  btn.className = 'elem-btn';
  btn.dataset.key = key;
  btn.innerHTML = `<span class="sw" style="background:${def.color}"></span>
                   <span class="nm">${def.corto}</span>
                   <span class="kbd">${i + 1}</span>`;
  btn.addEventListener('click', () => { initAudio(); sonidoClick(); seleccionar(key); });
  selector.appendChild(btn);
});
ORDEN_UI.forEach(key => {
  const def = ELEMENTOS[key];
  const s = document.createElement('span');
  s.innerHTML = `<i style="background:${def.color}"></i>${def.corto}`;
  leyenda.appendChild(s);
});

function seleccionar(key){
  const btn = document.querySelector(`.elem-btn[data-key="${key}"]`);
  if (btn && btn.classList.contains('cubicado')) return;
  selKey = key;
  const def = ELEMENTOS[key];
  document.querySelectorAll('.elem-btn').forEach(b =>
    b.classList.toggle('activo', b.dataset.key === key));
  lblLargo.textContent = def.etiquetas.largo + ' (m)';
  lblAncho.textContent = def.etiquetas.ancho + ' (m)';
  lblAlto.textContent  = def.etiquetas.alto  + ' (m)';
  [inLargo, inAncho, inAlto].forEach(i => i.classList.remove('ok','error'));
  inLargo.value = ''; inAncho.value = ''; inAlto.value = ''; inVol.value = '';
  actualizarFormulaViva();
  avisoDim.style.display = 'none';
  notaDetalle.style.display = (GRUPO_FUNDACION.indexOf(key) !== -1) ? 'block' : 'none';
  notaMuro.style.display    = (key === 'muro') ? 'block' : 'none';
  notaPilar.style.display   = (key === 'pilar') ? 'block' : 'none';
  hudElem.textContent = def.nombre;
  hudDims.textContent = 'Lee las cotas del modelo (m, cm o mm)';
  actualizarEtiquetas(); actualizarOpacidades(); mostrarCotasDe(key);
  if (estadoProyecto[key]) restaurarResultadoElemento(key);
  else limpiarResultados();
  centrarEnElemento(key, true);
}

function actualizarFormulaViva(){
  const L = parseFloat(inLargo.value);
  const A = parseFloat(inAncho.value);
  const H = parseFloat(inAlto.value);
  const ok = (n) => isFinite(n) && n > 0;
  const fmt = (n) => ok(n) ? n.toFixed(2) : '?';
  formulaViva.textContent = `V = ${fmt(L)} × ${fmt(A)} × ${fmt(H)} = ? m³`;
}
[inLargo, inAncho, inAlto].forEach(inp => {
  inp.addEventListener('input', () => {
    inp.classList.remove('ok','error');
    avisoDim.style.display = 'none';
    actualizarFormulaViva();
  });
});
document.getElementById('btn-vaciar').addEventListener('click', () => {
  sonidoClick();
  inLargo.value = ''; inAncho.value = ''; inAlto.value = '';
  [inLargo, inAncho, inAlto].forEach(i => i.classList.remove('ok','error'));
  avisoDim.style.display = 'none';
  actualizarFormulaViva(); inLargo.focus();
});

function limpiarResultados(){
  badge.className = 'badge'; badge.textContent = 'Sin calcular';
  rReal.textContent = '—'; rEst.textContent = '—';
  rDif.textContent = '—'; rPct.textContent = '—';
  rReal.className = 'v'; rEst.className = 'v'; rDif.className = 'v'; rPct.className = 'v';
  barraFill.style.width = '0%'; barraFill.className = '';
  pctLlenado.textContent = '0 %';
  feedback.className = 'feedback';
  feedback.innerHTML = 'Lee las cotas del modelo (m, cm o mm), conviértelas a metros, calcula el volumen y escríbelo.';
}

/* =========================================================
   16. VERTIDO
   ========================================================= */
let animacion = null;

function verter(){
  initAudio();
  if (estadoProyecto[selKey]){
    sonidoError();
    feedback.className = 'feedback warn';
    feedback.innerHTML = '🔒 Este elemento ya fue cubicado. Pulsa <b>Nueva partida</b> para reiniciar.';
    return;
  }
  const def = ELEMENTOS[selKey];

  /* 1) Validar dimensiones */
  const errores = validarDimensiones();
  if (errores.length > 0){
    sonidoError();
    avisoDim.style.display = 'block';
    avisoDim.innerHTML = '⚠️ <b>Revisa tu lectura de las cotas.</b><br>' +
      mensajeErrorDimensiones(errores);
    feedback.className = 'feedback bad';
    feedback.innerHTML = '❌ Las dimensiones ingresadas no coinciden con las cotas del modelo. Corrige los campos en rojo.';
    badge.className = 'badge bad'; badge.textContent = 'DIMENSIONES INCORRECTAS';
    return;
  }
  avisoDim.style.display = 'none';

  const { largo:L, ancho:A, alto:H } = def.dims;
  const real = volumenReal(selKey);
  const est = parseFloat(inVol.value);
  if (!isFinite(est) || est <= 0){
    sonidoError();
    feedback.className = 'feedback warn';
    feedback.innerHTML = '⚠️ Debes ingresar un <b>volumen total mayor que 0</b>.';
    inVol.focus(); return;
  }

  /* Capturar valores del usuario ANTES de cualquier limpieza */
  const dimsUsuario = {
    L: parseFloat(inLargo.value),
    A: parseFloat(inAncho.value),
    H: parseFloat(inAlto.value)
  };

  const diff = est - real;
  const pct  = (diff / real) * 100;
  const ratio = est / real;
  const fillFrac = Math.min(ratio, 1);
  const desborda = ratio > 1.002;

  let estado;
  if (Math.abs(pct) <= 2) estado = 'exacto';
  else if (diff < 0)      estado = 'falta';
  else                    estado = 'desborda';

  const cls = estado === 'exacto' ? 'ok' : (estado === 'falta' ? 'warn' : 'bad');
  const txt = estado === 'exacto' ? '✔ EXACTO' :
              estado === 'falta'  ? '▼ FALTA HORMIGÓN' : '▲ SE DESBORDA';
  badge.className = 'badge ' + cls; badge.textContent = txt;

  rReal.textContent = real.toFixed(3) + ' m³';
  rEst.textContent  = est.toFixed(3) + ' m³';
  rDif.textContent  = (diff >= 0 ? '+' : '') + diff.toFixed(3) + ' m³';
  rPct.textContent  = (pct >= 0 ? '+' : '') + pct.toFixed(2) + ' %';
  rDif.className = 'v ' + cls; rPct.className = 'v ' + cls;

  let extraMuro = '';
  if (selKey === 'muro' && def.vano){
    const v = def.vano;
    const vVano = v.ancho * v.alto * A;
    extraMuro = `<br><span style="color:#1e40af">Vano: ${v.ancho.toFixed(2)} × ${v.alto.toFixed(2)} m · a descontar: ${vVano.toFixed(3)} m³.</span>`;
  }

  let mensajeHTML = '';
  if (estado === 'exacto'){
    mensajeHTML =
      `<b>¡Cubicación correcta!</b> Tu volumen total está dentro del margen del 2&nbsp;%.<br>
       Real: ${real.toFixed(3)} m³ (cotas verificadas ✔).${extraMuro}`;
  } else if (estado === 'falta'){
    mensajeHTML =
      `<b>Falta hormigón.</b> Escribiste ${est.toFixed(3)} m³ pero el elemento necesita ${real.toFixed(3)} m³.<br>
       Te faltan <b>${Math.abs(diff).toFixed(3)} m³</b> (${Math.abs(pct).toFixed(2)}&nbsp;% menos).${extraMuro}`;
  } else {
    mensajeHTML =
      `<b>Se desborda.</b> Escribiste ${est.toFixed(3)} m³ y solo caben ${real.toFixed(3)} m³.<br>
       Sobran <b>${diff.toFixed(3)} m³</b> (${pct.toFixed(2)}&nbsp;% más).${extraMuro}`;
  }
  feedback.className = 'feedback ' + cls;
  feedback.innerHTML = mensajeHTML;

  /* ----- GAMIFICACIÓN ----- */
  juego.intentos++;
  let puntosGanados = 0;
  let estrellas = 0;
  let tipoLogro = estado === 'exacto' ? 'exacto' : (estado === 'falta' ? 'warn' : 'bad');
  let iconoLogro, tituloLogro, subtituloLogro;

  if (estado === 'exacto'){
    juego.aciertos++;
    juego.rachaActual++;
    if (juego.rachaActual > juego.mejorRacha) juego.mejorRacha = juego.rachaActual;

    const bonusRacha = Math.min(juego.rachaActual - 1, 4) * 25;
    puntosGanados = 100 + bonusRacha;

    const absPct = Math.abs(pct);
    if (absPct <= 0.5)      estrellas = 3;
    else if (absPct <= 1.25) estrellas = 2;
    else                    estrellas = 1;

    juego.estrellasTotales += estrellas;
    iconoLogro = estrellas === 3 ? '🏆' : '🎯';
    tituloLogro = estrellas === 3 ? '¡PERFECTO!' : '¡EXACTO!';
    subtituloLogro = `Error de ${absPct.toFixed(2)}% · ${estrellas} estrella${estrellas>1?'s':''}`;

    sonidoExacto();
    if (juego.rachaActual >= 3) setTimeout(sonidoVictoria, 500);

  } else if (estado === 'falta'){
    juego.rachaActual = 0;
    puntosGanados = 25;
    estrellas = 1;
    iconoLogro = '📉';
    tituloLogro = 'Falta hormigón';
    subtituloLogro = `${Math.abs(diff).toFixed(3)} m³ por debajo`;
    sonidoFalta();
  } else {
    juego.rachaActual = 0;
    puntosGanados = 25;
    estrellas = 1;
    iconoLogro = '💥';
    tituloLogro = 'Se desborda';
    subtituloLogro = `${diff.toFixed(3)} m³ de exceso`;
    sonidoDesborda();
  }

  juego.puntos += puntosGanados;
  juego.elementosCubicados++;
  actualizarHUDGamificacion();

  /* Guardar resultado */
  estadoProyecto[selKey] = {
    estimado: est, real, diff, pct, ratio, estado, cls, txt, mensaje: mensajeHTML,
    dims: { L, A, H },
    dimsUsuario
  };
  bloquearElemento(selKey, cls);
  actualizarPanelCostos();

  /* Animación del vertido */
  objetos[selKey].fillH = 0;
  objetos[selKey].concreto.visible = false;
  ocultarCharco();
  const dur = Math.min(5, Math.max(3.2, 3.2 + 1.0 * fillFrac + (desborda ? 0.8 : 0)));
  animacion = { key: selKey, t:0, dur, fillFrac, ratio, desborda, estado };
  barraFill.className = cls;

  /* Logro flotante */
  const esUltimo = (juego.elementosCubicados === ORDEN_UI.length);
  setTimeout(() => {
    if (esUltimo && !juego.proyectoCompletado){
      juego.proyectoCompletado = true;
      const bonusFinal = 150;
      juego.puntos += bonusFinal;
      actualizarHUDGamificacion();
      sonidoVictoria();
      mostrarLogro(
        'exacto', '🎉', '¡PROYECTO COMPLETADO!',
        `Sumaste ${juego.puntos} puntos · ${juego.estrellasTotales} ⭐`,
        3, bonusFinal,
        `<b>Balance del proyecto:</b><br>` +
        `• Aciertos exactos: <b>${juego.aciertos} / ${juego.intentos}</b><br>` +
        `• Mejor racha: <b>${juego.mejorRacha}</b><br>` +
        `• Puntos totales: <b>${juego.puntos}</b><br>` +
        `Pulsa el botón para ver el <b>informe técnico completo</b>.`,
        { confetti: true, textoBtn: '📄 Ver informe final', mostrarInforme: true }
      );
    } else {
      const bonusRachaTxt = (estado === 'exacto' && juego.rachaActual >= 2)
        ? `<br>🔥 Racha de <b>${juego.rachaActual}</b> aciertos · bonus aplicado.`
        : '';
      mostrarLogro(
        tipoLogro, iconoLogro, tituloLogro, subtituloLogro, estrellas, puntosGanados,
        mensajeHTML + bonusRachaTxt
      );
    }
  }, 900);
}

function actualizarAnimacion(dt){
  if (!animacion) return;
  const a = animacion;
  const def = ELEMENTOS[a.key];
  const H = def.dims.alto;
  a.t += dt;
  const p = Math.min(1, a.t / a.dur);
  const pFill = Math.min(1, p / 0.82);
  const e = pFill * pFill * (3 - 2 * pFill);
  const h = a.fillFrac * H * e;
  fijarRelleno(a.key, h);
  const objetivo = Math.min(a.ratio, 1.6) / 1.6;
  barraFill.style.width = (objetivo * e * 100).toFixed(1) + '%';
  pctLlenado.textContent = (e * a.ratio * 100).toFixed(1) + ' %';
  if (p < 1){
    mostrarFlujo(def, h);
    if (a.desborda && p > 0.35){
      spawnDesborde(def);
      if (p > 0.5) spawnGoterones(def);
      const exceso = Math.min(1.5, (a.ratio - 1) * 1.5);
      const prog = Math.min(1, (p - 0.35) / 0.55);
      mostrarCharco(def, prog, exceso);
    }
  } else {
    ocultarFlujo();
    pctLlenado.textContent = (a.ratio * 100).toFixed(1) + ' %';
    barraFill.style.width = (objetivo * 100).toFixed(1) + '%';
    if (a.desborda){
      const exceso = Math.min(1.5, (a.ratio - 1) * 1.5);
      mostrarCharco(def, 1, exceso);
    }
    animacion = null;
  }
}

/* =========================================================
   17. CONTROLES
   ========================================================= */
let arrastrando = false, px = 0, py = 0;
canvas.addEventListener('pointerdown', e => {
  arrastrando = true; px = e.clientX; py = e.clientY;
  canvas.setPointerCapture(e.pointerId);
});
canvas.addEventListener('pointerup', e => {
  arrastrando = false;
  try { canvas.releasePointerCapture(e.pointerId); } catch(_){}
});
canvas.addEventListener('pointermove', e => {
  if (!arrastrando) return;
  const dx = e.clientX - px, dy = e.clientY - py;
  px = e.clientX; py = e.clientY;
  theta -= dx * 0.008;
  phi -= dy * 0.006;
  phi = Math.max(0.18, Math.min(1.45, phi));
  actualizarCamara();
});
canvas.addEventListener('wheel', e => {
  e.preventDefault();
  frustum *= Math.exp(e.deltaY * 0.0011);
  frustum = Math.max(1.2, Math.min(35, frustum));
  camGoal.frustum = frustum;
  camCurrent.frustum = frustum;
  camLerp = 1;
  redimensionar();
}, { passive:false });

document.addEventListener('keydown', e => {
  if (e.target.tagName === 'INPUT') return;
  if (ach.el.classList.contains('show')){
    if (e.key === 'Enter' || e.key === 'Escape') { ocultarLogro(); e.preventDefault(); }
    return;
  }
  const n = parseInt(e.key, 10);
  if (n >= 1 && n <= 7){
    const k = ORDEN_UI[n - 1];
    const btn = document.querySelector(`.elem-btn[data-key="${k}"]`);
    if (!btn.classList.contains('cubicado')) { initAudio(); sonidoClick(); seleccionar(k); }
  }
  else if (e.key === 'Enter') verter();
  else if (e.key.toLowerCase() === 'c') centrarEnElemento(selKey, true);
});

/* Escape / Enter para cerrar el informe */
document.addEventListener('keydown', e => {
  if (document.getElementById('informe-final').classList.contains('show')){
    if (e.key === 'Escape'){ ocultarInforme(); e.preventDefault(); e.stopPropagation(); }
  }
}, true);

/* =========================================================
   18. RESIZE + LOOP
   ========================================================= */
function redimensionar(){
  const w = contenedor.clientWidth || 800;
  const h = contenedor.clientHeight || 600;
  const aspect = w / h;
  cam.left = -frustum * aspect / 2;
  cam.right =  frustum * aspect / 2;
  cam.top =  frustum / 2;
  cam.bottom = -frustum / 2;
  cam.updateProjectionMatrix();
  renderer.setSize(w, h, false);
  actualizarEscalaEtiquetas();
}
window.addEventListener('resize', redimensionar);

let tPrev = performance.now();
function loop(now){
  requestAnimationFrame(loop);
  const dt = Math.min(0.05, (now - tPrev) / 1000);
  tPrev = now;
  actualizarTransicionCamara(dt);
  actualizarAnimacion(dt);
  actualizarParticulas(dt);
  if (flujo.visible){
    texturaFlujo.offset.y -= dt * 1.9;
    flujo.rotation.y += dt * 0.6;
  }
  renderer.render(scene, cam);
}

/* =========================================================
   19. ARRANQUE
   ========================================================= */
document.getElementById('btn-verter').addEventListener('click', () => { initAudio(); verter(); });
document.getElementById('btn-centrar').addEventListener('click', () => { sonidoClick(); centrarEnElemento(selKey, true); });
document.getElementById('btn-centrar-hud').addEventListener('click', () => { sonidoClick(); centrarEnElemento(selKey, true); });

document.getElementById('btn-reset').addEventListener('click', () => {
  sonidoClick();
  ocultarInforme();
  animacion = null;
  for (const k of ORDEN_UI) delete estadoProyecto[k];
  desbloquearTodos();
  limpiarPanelCostos();
  limpiarRellenos();
  limpiarResultados();
  inVol.value = ''; inLargo.value = ''; inAncho.value = ''; inAlto.value = '';
  [inLargo, inAncho, inAlto].forEach(i => i.classList.remove('ok','error'));
  avisoDim.style.display = 'none';
  reiniciarGamificacion();
  randomizarDimensiones();
  actualizarTodo();
  seleccionar('muro');
});

/* Botón de informe permanente */
document.getElementById('btn-informe').addEventListener('click', () => {
  initAudio(); sonidoClick(); mostrarInforme();
});

/* Toggle sonido */
const btnSound = document.getElementById('btn-sound');
btnSound.addEventListener('click', () => {
  sonidoActivo = !sonidoActivo;
  btnSound.textContent = sonidoActivo ? '🔊' : '🔇';
  btnSound.title = sonidoActivo ? 'Silenciar' : 'Activar sonido';
  if (sonidoActivo){ initAudio(); sonidoClick(); }
});

/* --- Arranque --- */
randomizarDimensiones();
actualizarCamara();
redimensionar();
actualizarTodo();
seleccionar('muro');
limpiarPanelCostos();
actualizarFormulaViva();
actualizarHUDGamificacion();
requestAnimationFrame(loop);
</script>
</body>
</html>
