<template>

    <header>
        <div style="padding: 18px 24px;">
            <div style="display: flex; align-items: center;">
                <div>
                    <h1 style="font-family: 'Audiowide', 'serif'; font-weight: 400; font-style: normal; color: #03AC0E; font-size: 1.875rem; margin: 0;">Kedaipedia</h1>
                </div>
                <div style="flex-grow: 1; display: flex; justify-content: center;">
                    <div style="width: 70%;">
                        <div style="border: 1px solid #bfc9d9; padding: 9px 12px; border-radius: 8px; display: flex; gap: 6px; align-items: center;">
                            <svg viewBox="0 0 24 24" width="24" height="24" fill="#8D96AA">
                                <path d="M20.53 19.46l-4.4-4.4a7.33 7.33 0 10-1.07 1.06l4.41 4.41a.77.77 0 001.06 0 .77.77 0 000-1.07zm-15.78-9a5.75 5.75 0 115.75 5.75 5.76 5.76 0 01-5.75-5.72v-.03z"></path>
                            </svg>
                            <input type="text" style="width: 100%; background: none; border: none; font-size: 0.875rem; color: #75797E; outline: none;" placeholder="Cari Nama Barang / Nama Toko ...">
                        </div>
                    </div>
                </div>
                <div>
                    <button style="background: none; border: 1px solid #03AC0E; color: #03AC0E; border-radius: 7px; margin: 0; padding: 4px 12px; font-size: 0.875rem; font-weight: 700;">
                        Masuk
                    </button>
                </div>
            </div>
        </div>
    </header>
    
</template>

<style>

header {
    width: 100%;
    background-color: white;
    position: sticky;
    top: 0;
    box-shadow: rgba(0, 0, 0, 0.07) 0px 4px 6px -1px;
    z-index: 10;
}

input:focus {
    border: none;
}

</style>