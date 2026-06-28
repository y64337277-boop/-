# -
مشروع يعرف كيف نكتب الحروف والارقام بلغة الكمبيوتر الالة
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import 'dart:convert';
import 'package:screenshot/screenshot.dart';
import 'package:image_gallery_saver/image_gallery_saver.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:flutter_tts/flutter_tts.dart'; // 1. استيراد TTS

void main() {
  runApp(const BinaryApp());
}

class BinaryApp extends StatelessWidget {
  const BinaryApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'من 0 و 1 إلى الكود',
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.indigo,
        textTheme: GoogleFonts.firaCodeTextTheme(),
      ),
      home: const HomeScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> with SingleTickerProviderStateMixin {
  late TabController _tabController;
  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 4, vsync: this);
  }
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('من 0 و 1 إلى الكود'),
        bottom: TabBar(
          isScrollable: true,
          controller: _tabController,
          tabs: const [
            Tab(text: '1. Binary', icon: Icon(Icons.code)),
            Tab(text: '2. Assembly', icon: Icon(Icons.memory)),
            Tab(text: '3. C/Python', icon: Icon(Icons.terminal)),
            Tab(text: '4. الذاكرة', icon: Icon(Icons.storage)),
          ],
        ),
      ),
      body: TabBarView(
        controller: _tabController,
        children: const [
          BinaryScreen(),
          AssemblyScreen(),
          HighLevelScreen(),
          MemoryScreen(),
        ],
      ),
    );
  }
}

// ==================== منطق التحويل ====================
class ByteInfo {
  final String char;
  final int decimal;
  final String binary;
  final int address;
  ByteInfo({required this.char, required this.decimal, required this.binary, required this.address});
}

class Converter {
  static List<ByteInfo> textToBytes(String text) {
    List<int> bytes = utf8.encode(text);
    List<ByteInfo> result = [];
    int baseAddress = 0x1000;
    for (int i = 0; i < bytes.length; i++) {
      String char = i < text.length? text[i] : '';
      result.add(ByteInfo(
        char: char,
        decimal: bytes[i],
        binary: bytes[i].toRadixString(2).padLeft(8, '0'),
        address: baseAddress + i,
      ));
    }
    return result;
  }
  static Map<String, String> textToHighLevel(String text) {
    List<ByteInfo> bytes = textToBytes(text);
    String cCode = 'char name[] = "$text";\n\n// في الذاكرة:\n';
    for (var b in bytes) { cCode += 'name[${bytes.indexOf(b)}] = ${b.decimal}; // ${b.char} = ${b.binary}\n'; }
    cCode += 'name[${bytes.length}] = 0; // \\0';
    String pyCode = 'name = "$text"\n\n# في الذاكرة:\n';
    for (var b in bytes) { pyCode += 'byte_${bytes.indexOf(b)} = ${b.decimal} # ${b.char} = ${b.binary}\n'; }
    return {'C': cCode, 'Python': pyCode};
  }
}

//... شاشات 1 و 2 و 3 نفسها بدون تغيير...
class BinaryScreen extends StatefulWidget { const BinaryScreen({super.key}); @override State<BinaryScreen> createState() => _BinaryScreenState(); }
class _BinaryScreenState extends State<BinaryScreen> {
  final _text = TextEditingController(); final _binary = TextEditingController(); String _details = '';
  void _toBinary() { var bytes = Converter.textToBytes(_text.text); setState(() { _binary.text = bytes.map((b) => b.binary).join(' '); _details = bytes.map((b) => '${b.binary} -> ${b.char}').join('\n'); }); }
  @override Widget build(BuildContext context) { return Padding(padding: const EdgeInsets.all(16.0), child: Column(children: [TextField(controller: _text, decoration: const InputDecoration(labelText: 'النص', border: OutlineInputBorder())), const SizedBox(height: 12), TextField(controller: _binary, decoration: const InputDecoration(labelText: 'Binary', border: OutlineInputBorder())), const SizedBox(height: 16), FilledButton(onPressed: _toBinary, child: const Text('نص -> 0 و 1')), const SizedBox(height: 20), Expanded(child: Container(width: double.infinity, padding: const EdgeInsets.all(12), decoration: BoxDecoration(color: Colors.grey[100], borderRadius: BorderRadius.circular(12)), child: SingleChildScrollView(child: Text(_details, style: const TextStyle(fontSize: 16))))) ])); }
}
class AssemblyScreen extends StatefulWidget { const AssemblyScreen({super.key}); @override State<AssemblyScreen> createState() => _AssemblyScreenState(); }
class _AssemblyScreenState extends State<AssemblyScreen> { final _text = TextEditingController(); String _explain = ''; void _toAsm() { var bytes = Converter.textToBytes(_text.text); String allAsm = bytes.map((b) => 'MOV AL, ${b.decimal} ; ${b.char} = ${b.binary}').join('\n'); setState(() => _explain = allAsm); } @override Widget build(BuildContext context) { return Padding(padding: const EdgeInsets.all(16.0), child: Column(children: [TextField(controller: _text, decoration: const InputDecoration(labelText: 'النص', hintText: 'Ali', border: OutlineInputBorder())), const SizedBox(height: 16), FilledButton(onPressed: _toAsm, child: const Text('حول لـ Assembly')), const SizedBox(height: 20), Expanded(child: Container(width: double.infinity, padding: const EdgeInsets.all(12), decoration: BoxDecoration(color: Colors.black87, borderRadius: BorderRadius.circular(12)), child: SingleChildScrollView(child: Text(_explain, style: GoogleFonts.firaCode(color: Colors.greenAccent, fontSize: 15)))))])); }
}
class HighLevelScreen extends StatefulWidget { const HighLevelScreen({super.key}); @override State<HighLevelScreen> createState() => _HighLevelScreenState(); }
class _HighLevelScreenState extends State<HighLevelScreen> { final _text = TextEditingController(); String _cCode = ''; String _pyCode = ''; void _generateCode() { var codes = Converter.textToHighLevel(_text.text); setState(() { _cCode = codes['C']!; _pyCode = codes['Python']!; }); } @override Widget build(BuildContext context) { return Padding(padding: const EdgeInsets.all(16.0), child: Column(children: [TextField(controller: _text, decoration: const InputDecoration(labelText: 'اكتب اسمك', hintText: 'Ali', border: OutlineInputBorder())), const SizedBox(height: 16), FilledButton(onPressed: _generateCode, child: const Text('اظهر كود C و Python')), const SizedBox(height: 20), Expanded(child: SingleChildScrollView(child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [Text('C Code:', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)), Container(width: double.infinity, padding: const EdgeInsets.all(12), margin: const EdgeInsets.only(top: 8, bottom: 16), decoration: BoxDecoration(color: Colors.blueGrey[900], borderRadius: BorderRadius.circular(12)), child: Text(_cCode, style: GoogleFonts.firaCode(color: Colors.lightBlueAccent, fontSize: 14))), Text('Python Code:', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)), Container(width: double.infinity, padding: const EdgeInsets.all(12), margin: const EdgeInsets.only(top: 8), decoration: BoxDecoration(color: Colors.amber[900], borderRadius: BorderRadius.circular(12)), child: Text(_pyCode, style: GoogleFonts.firaCode(color: Colors.yellow[100], fontSize: 14))) ])))])); }
}

// ==================== شاشة 4: رسم الذاكرة + تصدير + صوت ====================
class MemoryScreen extends StatefulWidget {
  const MemoryScreen({super.key});
  @override
  State<MemoryScreen> createState() => _MemoryScreenState();
}
class _MemoryScreenState extends State<MemoryScreen> {
  final _text = TextEditingController();
  List<ByteInfo> _bytes = [];
  final ScreenshotController screenshotController = ScreenshotController();
  final FlutterTts flutterTts = FlutterTts(); // 2. كنترولر الصوت

  @override
  void initState() {
    super.initState();
    _initTts();
  }

  Future<void> _initTts() async {
    await flutterTts.setLanguage("ar-SA"); // عربي سعودي
    await flutterTts.setSpeechRate(0.5); // سرعة نص
  }

  void _drawMemory() {
    setState(() { _bytes = Converter.textToBytes(_text.text); });
  }

  Future<void> _speakByte(ByteInfo b) async {
    if (b.char == '\\0') {
      await flutterTts.speak("نهاية النص صفر");
      return;
    }
    String speech = "الحرف ${b.char} قيمته ${b.decimal} باينري ${b.binary.split('').join(' ')}";
    await flutterTts.speak(speech);
  }

  Future<void> _exportImage() async {
    if (_bytes.isEmpty) return;
    var status = await Permission.storage.request();
    if (!status.isGranted) return;
    final image = await screenshotController.capture();
    if (image == null) return;
    await ImageGallerySaver.saveImage(image, name: "memory_${_text.text}_${DateTime.now().millisecondsSinceEpoch}");
    if (!mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('تم حفظ الصورة في المعرض ✅')));
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Column(children: [
        TextField(controller: _text, decoration: const InputDecoration(labelText: 'اكتب اسمك لترى الذاكرة', hintText: 'Ali', border: OutlineInputBorder())),
        const SizedBox(height: 16),
        Row(children: [
          Expanded(child: FilledButton.icon(onPressed: _drawMemory, icon: const Icon(Icons.draw), label: const Text('ارسم الذاكرة'))),
          const SizedBox(width: 8),
          Expanded(child: OutlinedButton.icon(onPressed: _exportImage, icon: const Icon(Icons.download), label: const Text('تصدير صورة'))),
        ]),
        const SizedBox(height: 20),
        Expanded(
          child: Screenshot(
            controller: screenshotController,
            child: Container(
              color: Theme.of(context).scaffoldBackgroundColor,
              child: _bytes.isEmpty
               ? const Center(child: Text('ادخل اسم واضغط ارسم'))
                  : GridView.builder(
                      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 2, childAspectRatio: 1.8, crossAxisSpacing: 10, mainAxisSpacing: 10),
                      itemCount: _bytes.length + 1,
                      itemBuilder: (context, index) {
                        if (index == _bytes.length) {
                          return _buildByteCard(ByteInfo(char: '\\0', decimal: 0, binary: '00000', address: _bytes.last.address + 1), isNull: true);
                        }
                        return _buildByteCard(_bytes[index]);
                      },
                    ),
            ),
          ),
        )
      ]),
    );
  }

  Widget _buildByteCard(ByteInfo b, {bool isNull = false}) {
    return InkWell( // 3. خلي الكرت قابل للضغط
      onTap: () => _speakByte(b),
      child: Card(
        elevation: 2,
        color: isNull? Colors.red[100] : Colors.indigo[50],
        child: Padding(
          padding: const EdgeInsets.all(8.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('0x${b.address.toRadixString(16).toUpperCase()}', style: const TextStyle(fontSize: 12, color: Colors.grey)),
              const Divider(),
              Text(b.char.isEmpty? '?' : b.char, style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
              const SizedBox(height: 4),
              Text('${b.decimal}', style: const TextStyle(fontSize: 14)),
              Text(b.binary, style: const TextStyle(fontSize: 11, fontFamily: 'monospace')),
              const Icon(Icons.volume_up, size: 16, color: Colors.grey), // ايقونة صوت
            ],
          ),
        ),
      ),
    );
  }
}
