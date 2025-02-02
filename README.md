SELECT
  -- Informasi transaksi
  transaction_id,
  date,
  -- Informasi cabang
  branch_id,
  branch_name,
  kota,
  provinsi,
  -- Informasi customer
  customer_name,
  -- Informasi produk
  product_id,
  product_name,
  actual_price,
  discount_percentage,
  -- Metrics keuangan
  persentase_gross_laba,
  nett_sales,
  -- Menghitung keuntungan bersih (pembulatan 2 desimal)
  ROUND(nett_sales * (persentase_gross_laba / 100), 2) AS nett_profit,
  -- Rating transaksi
  rating_transaksi
FROM (
  SELECT
    ft.transaction_id,
    ft.date,
    kc.branch_id,
    kc.branch_name,
    kc.kota,
    kc.provinsi,
    ft.customer_name,
    p.product_id,
    p.product_name,
    p.price AS actual_price,
    ft.discount_percentage,

    -- Persentase laba kotor berdasarkan harga
    CASE
      WHEN p.price <= 50000 THEN 10
      WHEN p.price <= 100000 THEN 15
      WHEN p.price <= 300000 THEN 20
      WHEN p.price <= 500000 THEN 25
      ELSE 30
    END AS persentase_gross_laba,

    -- Menghitung penjualan bersih (harga setelah diskon)
    p.price * (1 - ft.discount_percentage / 100) AS nett_sales,

    -- Rating transaksi
    ft.rating AS rating_transaksi
  FROM 
    kimia-farma-data-anlysis.Kimia_farma.kf_final_transaction ft
    JOIN kimia-farma-data-anlysis.Kimia_farma.kf_kantor_cabang kc ON ft.branch_id = kc.branch_id
    JOIN kimia-farma-data-anlysis.Kimia_farma.kf_product p ON ft.product_id = p.product_id
  WHERE 
    ft.date >= '2020-01-01' AND ft.date <= '2023-12-31' -- Filter data tahun 2020-2023
) AS profit_calculation
ORDER BY 
  date DESC,
  transaction_id DESC;
