---
title: xlsx修改加密文件
tags:
  - xlsx
categories:
  - xlsx
mathjax: true
description: xlsx
abbrlink: 4051d6c5
date: 2025-02-17 19:04:45
---

### xlsx解密修改文件内容并重新加密下载

```tsx
import {
    ZipReader,
    BlobReader,
    TextWriter,
    ZipWriter,
    BlobWriter
} from '@zip.js/zip.js';
import XLSX from 'xlsx';

class KeepZero {
    public getKeepZeroXlsx = (blob, password, path) => {
        this.extractAndDecryptXlsx(blob, password, path).then(zipContent => {
            if (zipContent) {
                console.log('解密成功，文件已经处理', zipContent);
                const xlsxBlob = this.convertToXLSX(zipContent);
                console.log('xlsxBlob', xlsxBlob);
                // this.readAndPrintXLSX(xlsxBlob);
                const nowBolb = this.encryptAndDownload(xlsxBlob, password, path);
                // console.log('nowBolb', nowBolb);
                // 你可以在这里进一步操作解压后的文件内容
            }
        });
    };

    // 解压 ZIP，提取并解密 .xlsx 文件
    public extractAndDecryptXlsx = async (blob, password, path) => {
        try {
            // 创建 ZipReader 实例，提供密码
            const reader = new ZipReader(new BlobReader(blob), { password });
            // 获取文件条目
            const entries = await reader.getEntries();
            
            if (entries.length === 1 && !entries[0].directory) {
                // 读取文件内容为文本
                const content = await entries[0].getData?.(new TextWriter());
                // 关闭读取器
                await reader.close();
                return { filename: path, content }; // 返回文件名和内容
            } else {
                console.error('ZIP 文件包含多个文件或没有文件');
                return null;
            }
        } catch (err) {
            console.error('解密失败:', err);
            return null;
        }
    };

    public convertToXLSX = (fileContent) => {
        console.log('XLSX?.utils', XLSX.utils);
        // 创建一个新的工作簿
        const workbook = XLSX.utils.book_new();
        // 假设内容是 CSV 格式，按逗号分隔每行
        const csvData = fileContent.content.split('\n').map(line => line.split(','));
        // 创建一个工作表
        const worksheet = XLSX.utils.aoa_to_sheet(csvData);
        
        // 遍历工作表数据，强制将某些列作为文本处理以保留前导零
        Object.keys(worksheet).forEach(cellAddress => {
            const cell = worksheet[cellAddress];
            if (cell && typeof cell.v === 'string' && /^[0-9]+$/.test(cell.v)) {
                // 如果单元格的值是纯数字字符串，则设置为文本格式
                cell.z = '@'; // 设置单元格为文本格式
            }
        });
        
        // 将工作表添加到工作簿
        XLSX?.utils?.book_append_sheet(workbook, worksheet, 'Sheet1');
        // 导出为 XLSX 文件
        const xlsxArray = XLSX?.write?.(workbook, { bookType: 'xlsx', type: 'array' });
        // 使用 Blob 构造函数将数组转换为 Blob
        const xlsxBlob = new Blob([xlsxArray], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        return xlsxBlob; // 返回生成的 Blob 对象
    };

    // 读取生成的 XLSX 文件并打印内容
    public readAndPrintXLSX = (xlsxBlob) => {
        // 创建 FileReader 实例读取 Blob
        const reader = new FileReader();
        reader.onload = function(e) {
            const data = new Uint8Array(e.target.result);
            const workbook = XLSX.read(data, { type: 'array' });
            // 获取第一个工作表
            const firstSheetName = workbook.SheetNames[0];
            const worksheet = workbook.Sheets[firstSheetName];
            // 将工作表内容转换为 JSON 格式
            const sheetData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
            console.log('打印 XLSX 内容:', sheetData); // 打印内容
        };
        reader.readAsArrayBuffer(xlsxBlob);
    };

    // 使用 @zip.js/zip.js 加密生成的 XLSX 文件并下载
    public encryptAndDownload = async (xlsxBlob, password, path) => {
        try {
            // 创建 ZipWriter 实例
            const zipWriter = new ZipWriter(new BlobWriter('application/zip'), { password });
            // 将 XLSX 文件添加到 ZIP 包中
            await zipWriter.add(`${path}.xlsx`, new BlobReader(xlsxBlob));
            // 关闭并获取加密后的 ZIP Blob
            const encryptedZipBlob = await zipWriter.close();
            
            // 创建下载链接
            const url = URL.createObjectURL(encryptedZipBlob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `${path || ''}.zip`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url); // 释放 URL
        } catch (err) {
            console.error('加密失败:', err);
        }
    };
}

// 实例化类并调用方法
const keepZero = new KeepZero();
const name = action.payload.excelName;
const realName = name.slice(-4) === '.zip' ? name.substring(0, name.length - 4) : name;
console.log('realName', realName);
keepZero.getKeepZeroXlsx(res.data, '123455', realName);

```

